# 📡 실시간 공지사항 팝업 만들기

이번에 만들 앱은, firebase의 remote config와 A_B Test를 이용해 실시간 공지사항 팝업을 만들어보려 한다.

우리가 만약 어떤 기업에 들어갔는데

디자이너 한 분이

### 🤷🏻‍♀️ “이번에 기획한 새 피처가 있는데요, A안이 나을지 B안이 나을지 가설 검증이 필요해요! 사용자에게 둘 다 보여주고 싶어요!”

라고 하거나, 또는 마케터 분이 

### 🙋🏻‍♂️ “이벤트 페이지에 마케팅을 돌릴건데 사용자 나이별로 후킹하는 마케팅 문구가 달라요! 타겟별로 다른 문구가 보이게 개발해주세요!”

라고 하거나, 또는 기획자 님이

### 🤦🏼‍♂️ “서버 안정화 작업으로 서비스 중단 공지팝업이 필요한데 완료 시간이 계속 바뀔 수 있어요. 수정하고 심사받아서 배포하려면 최소 하루는 걸릴텐데 어떡하죠?! 배포 없이 UI 수정 안되나요?”

라고 울먹인다면 여러분은 과연 어떻게 할 것인가?

바로 이때 필요한 것이 Firebase의 원격 구성, ***remote config와 A_B Test***이다.

우리는 이번 앱에서 디자이너 분이 말씀하신대로 alert A안 B안을 선택적으로 보이게 할 것이고.

기획자 분이 말씀하신대로 몇 분 내로 내용을 업데이트 할 수 있는 공지사항 팝업도 만들어 볼 예정이다.

## Firebase Remote Config 알아보기

원격 구성은 

- 배포없이, 업데이트 다운로드 없이 앱 변경
- 기본값 설정 후 값 재정의
- 클라우드 기반 key-value 저장소

주요기능

- 앱 사용자층에 변경사항을 빠르게 적용 - 업데이트 없이 앱의 UI/UX 변경 지원
- 사용자층의 특정 세그먼트에 앱 맞춤설정 - 앱 버전, 언어 등으로 분류된 사용자 세그먼트별 환경 제공
- A/B 테스트를 실행하여 앱 개선 - 사용자 세그먼트별로 개선사항을 검증 후 점진적 적용

## Firebase A/B Testing 알아보기

- Google Analytics, Firebase 예측을 통한 사용자 타겟팅
- 원격 구성(Remote Config) 또는 알림작성기(Cloud Messaging) 활용
- 제품, 마케팅 실험을 쉽게 실행, 분석, 확장
- 예를 들어서 특정 마케팅 캐릭터의 색에 관해 사용자의 의견이 갈렸을 때, 서로 다른 색 적용 가능

주요기능

- 제품 환경 테스트 및 개선 - 앱 동작 및 모양을 변경하여 최적의 제품 환경 확인
- 사용자의 재참여를 유도할 방안 모색 - 앱 사용자를 늘리기에 가장 효과적인 문구와 메시징 설정
- 새로운 기능의 안전한 구현 - 작은 규모의 사용자 집합을 대상으로 원하는 목표를 달성할 수 있는지 확인
- 예측된 사용자 그룹 타겟팅 - 특정 행동을 할 것으로 예측된 사용자에 A/B 테스트를 실시

이제 진짜 앱 만들기

만든 순서 

NoticeViewController 제작 → NoticeViewController.xib에서 notice 뷰 구성
<img width="1402" alt="스크린샷 2023-12-26 오후 1 10 52" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/cb375071-640b-450c-9e52-895f7aa9bdd1">
라벨도 연결(각 라벨의 텍스트를 원격 구성에서 받아온 정보로 표시)

```swift
//
//  NoticeViewController.swift
//  Notice
//
//  Created by jinyong yun on 12/26/23.
//

import UIKit

class NoticeViewController: UIViewController {
    var noticeContents: (title: String, detail: String, date: String)?

    @IBOutlet weak var noticeView: UIView!
    
    @IBOutlet weak var titleLabel: UILabel!
    
    @IBOutlet weak var detailLabel: UILabel!
    
    @IBOutlet weak var dateLabel: UILabel!

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        noticeView.layer.cornerRadius = 6
        view.backgroundColor = UIColor.black.withAlphaComponent(0.5)
        
        guard let noticeContents = noticeContents else { return }
        
        titleLabel.text = noticeContents.title
        detailLabel.text = noticeContents.detail
        dateLabel.text = noticeContents.date
    }

    @IBAction func doneButtonTapped(_ sender: UIButton) {
        self.dismiss(animated: true, completion: nil)
    }
    

}
```

이제 기본 UI 구성은 끝!

**Firebase 연결하기**

프로젝트 시작하기(이름은 notice) → Google 애널리틱스 활용할 예정이라 활성화 상태로 →프로젝트 만들기 끝

iOS 앱 추가 → 번들 ID 입력 → plist 다운로드 후 프로젝트에 추가!! → pod init → firebase 추가 

pod 'Firebase/RemoteConfig'
pod 'Firebase/Analytics'

(이 두가지로 원격구성과 A_B 둘 다 사용가능)

→ pod install → AppDelegate에서 초기화 (import Firebase 그리고 didFinishLaunchWithOptions에서 FirebaseApp.configure() 추가)

## Remote Config로 팝업 제어하기

원격구성은 Key-value 형태로 설정하며, 기본값을 업데이트 하는 방식이라고 앞서 설명했다.

우리는 ViewController에서 원격구성값을 불러와서 NoticeViewController에 내용을 전달하고 제어하도록 해보겠다.

먼저 기본 ViewController에 원격구성 플랫폼을 가져와야한다.

```swift
//
//  ViewController.swift
//  Notice
//
//  Created by jinyong yun on 12/26/23.
//

import UIKit
import FirebaseRemoteConfig

class ViewController: UIViewController {

    var remoteConfig: RemoteConfig?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        remoteConfig = RemoteConfig.remoteConfig()
        
        let setting = RemoteConfigSettings()
        setting.minimumFetchInterval = 0 //테스트를 위해 새로운 값을 패치하는 인터벌 최소화
        
        remoteConfig?.configSettings = setting
    }

}
```

remoteConfig 세팅까지 완료했다면 각 키의 기본값을 설정해줘야 하는데, 

plist를 하나 추가해준다.
<img width="1053" alt="스크린샷 2023-12-26 오후 1 44 43" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/8c314a69-1d02-458c-a703-a5e66b542026">
이렇게 plist를 인식할 수 있도록 ViewController의 viewDidLaod에 다음과 같은 문구 추가

```swift
remoteConfig?.setDefaults(fromPlist: "RemoteConfigDefaults")
```

이제 Firebase 콘솔로 이동 → 참여에서 remoteConfig 선택 → 구성 만들기 → 매개변수 추가
<img width="1375" alt="스크린샷 2023-12-26 오후 1 47 13" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/bd441198-e6e5-4499-b6b6-01e1737ce6e3">
<img width="1387" alt="스크린샷 2023-12-26 오후 1 49 16" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/abf52bed-8b08-45a9-af08-502884a382ad">
xcode로 돌아와 콘솔에 설정할 값들을 패칭

```swift
//RemoteConfig
extension ViewController {
    func getNotice() {
        guard let remoteConfig = remoteConfig else {return}
        remoteConfig.fetch { [weak self] status, _ in
            if status == .success {
                remoteConfig.activate(completion: nil)
            } else {
              print("ERROR: Config not fetched")
            }
            
            guard let self = self else {return}
            
            if !self.isNoticeHidden(remoteConfig) {
                let noticeVC = NoticeViewController(nibName: "NoticeViewController", bundle: nil)
                noticeVC.modalPresentationStyle = .custom
                noticeVC.modalTransitionStyle = .crossDissolve
                
                let title = (remoteConfig["title"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n") //firebase 콘솔 매개변수에서 여러 줄의 string을 넣으면 패치에서 \n하는 과정 중 \\n이렇게 역슬래쉬가 2번 찍힘
                let detail = (remoteConfig["detail"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n")
                let date = (remoteConfig["date"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n")
                
                noticeVC.noticeContents = (title: title, detail: detail, date: date)
                self.present(noticeVC, animated: true, completion: nil)
            }
        }
    }
    
    func isNoticeHidden(_ remoteConfig: RemoteConfig) -> Bool {
        return remoteConfig["isHidden"].boolValue
    }
}
```

Firebase 콘솔에서 기본값을 업데이트 해주고(xcode plist에 설정된 기본값과 같아서…)
<img width="1391" alt="스크린샷 2023-12-26 오후 2 03 17" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/8838985e-fe57-4d29-ab85-2c42e0a2d61b">

마지막으로 viewWillAppear에 만들어준 함수를 넣어줘야 하겠다

```swift
//
//  ViewController.swift
//  Notice
//
//  Created by jinyong yun on 12/26/23.
//

import UIKit
import FirebaseRemoteConfig

class ViewController: UIViewController {

    var remoteConfig: RemoteConfig?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        remoteConfig = RemoteConfig.remoteConfig()
        
        let setting = RemoteConfigSettings()
        setting.minimumFetchInterval = 0 //테스트를 위해 새로운 값을 패치하는 인터벌 최소화
        
        remoteConfig?.configSettings = setting
        remoteConfig?.setDefaults(fromPlist: "RemoteConfigDefaults")
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        getNotice()
    }

}

//RemoteConfig
extension ViewController {
    func getNotice() {
        guard let remoteConfig = remoteConfig else {return}
        remoteConfig.fetch { [weak self] status, _ in
            if status == .success {
                remoteConfig.activate(completion: nil)
            } else {
              print("ERROR: Config not fetched")
            }
            
            guard let self = self else {return}
            
            if !self.isNoticeHidden(remoteConfig) {
                let noticeVC = NoticeViewController(nibName: "NoticeViewController", bundle: nil)
                noticeVC.modalPresentationStyle = .custom
                noticeVC.modalTransitionStyle = .crossDissolve
                
                let title = (remoteConfig["title"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n") //firebase 콘솔 매개변수에서 여러 줄의 string을 넣으면 패치에서 \n하는 과정 중 \\n이렇게 역슬래쉬가 2번 찍힘
                let detail = (remoteConfig["detail"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n")
                let date = (remoteConfig["date"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n")
                
                noticeVC.noticeContents = (title: title, detail: detail, date: date)
                self.present(noticeVC, animated: true, completion: nil)
            }
        }
    }
    
    func isNoticeHidden(_ remoteConfig: RemoteConfig) -> Bool {
        return remoteConfig["isHidden"].boolValue
    }
}
```

위는 전체 코드
<img width="1677" alt="스크린샷 2023-12-26 오후 2 08 34" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/0244adde-06f6-4f49-9327-cd354d4e2bc8">
기본값이 아닌 서버에 만들어준 기본값이 나오는 것을 알 수 있다!!


https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/f4e00344-586f-438a-91fa-2b5fc03abedc
Firebase 콘솔에서 매개변수 수정을 누르고, 새로 추가 → 조건부 값 → 새 조건 만들기를 누르고

새로운 조건을 지정하여 해당 매개변수가 다르게 나타나게 할 수 있다.
<img width="1366" alt="스크린샷 2023-12-26 오후 2 12 32" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/db55acbd-88f7-4a49-87d5-0aae86888100">
## A/B Test로 팝업 제어하기

이제 별도 공지사항이 없을 경우 앱 사용이 원할할 때 이벤트 사용을 보여줄 것이다.

이 이벤트 알럿에 어떤 문구를 넣었을 때 더 사용자가 참여를 잘할 지 등에 사용할 수 있다.

먼저 Firebase 콘솔로 이동하여 A/B Testing을 만들어주자
<img width="1381" alt="스크린샷 2023-12-26 오후 7 27 59" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/7cc5c0a1-3d01-4296-9a60-0d1553c84eda">
A/B Testing을 만들어주고, 실험 만들기 후 원격 구성을 클릭한다.

그럼 원격 구성 실험 만들기 화면이 뜨는데, 여기서 테스트 이름과 설명을 [기본사항]에서 적고

[타겟팅]에서 대상 사용자를 설정한다. 앱을 선택하고(우리는 notice 앱) 대상 사용자의 50%가 이 실험에 노출되도록 설정한다. [목표]에서는 얼마나 앱에 들어오는지, 수익률은 얼마인지를 확인할 수 있다. 또 우리가 자체적으로 생성한 이벤트를 추적하도록 할 수 있다. 우리는 이벤트 문구를 무엇으로 설정했을 때 더 많이 확인하는지, 즉 확인 버튼을 언제 더 많이 누르는 지 확인하고 싶다. 그래서 새 이벤트를 만들건데

측정항목 추가에서 새 이벤트 만들기에 들어가 새로운 이벤트를 만들어준다. 이름은 promotion_alert으로 설정해줬다.

[변형]에서는 값 설정을 해줘야 한다. 즉 어떤 값과 어떤 값을 비교할 지를 설정해줘야 한다.
<img width="1360" alt="스크린샷 2023-12-26 오후 7 34 26" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/687553d5-80a0-4ee1-bf19-00b2c029fee9">

값을 설정했다면 검토를 누르고, 실험 시작을 눌러준다.

다시 원격 구성 화면으로 돌아가면 기존에 없었던 message라는 매개변수가 새로 생긴 것이 보인다.
<img width="1068" alt="스크린샷 2023-12-26 오후 7 35 41" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/0592db8f-6fed-4be6-8606-28abf0811906">
이제 실제 실험을 해보기 위해 코드로 설정해보자

우리는 위에서 만들었던 공지사항이 없을 때, 즉 공지사항 팝업이 뜨지 않을 때 이벤트 알람을 보여주려고 한다.

```swift

if !self.isNoticeHidden(remoteConfig) {
                let noticeVC = NoticeViewController(nibName: "NoticeViewController", bundle: nil)
                noticeVC.modalPresentationStyle = .custom
                noticeVC.modalTransitionStyle = .crossDissolve
                
                let title = (remoteConfig["title"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n") //firebase 콘솔 매개변수에서 여러 줄의 string을 넣으면 패치에서 \n하는 과정 중 \\n이렇게 역슬래쉬가 2번 찍힘
                let detail = (remoteConfig["detail"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n")
                let date = (remoteConfig["date"].stringValue ?? "").replacingOccurrences(of: "\\n", with: "\n")
                
                noticeVC.noticeContents = (title: title, detail: detail, date: date)
                self.present(noticeVC, animated: true, completion: nil)
            } else {
                self.showEventAlert()
            }
```

getNotice 함수에서 remoteConfig의 isNoticeHidden이 없을 때, showEventAlert()을 실행하도록 하고

```swift
//A/B Testing
extension ViewController {
    func showEventAlert() {
        guard let remoteConfig = remoteConfig else { return }
        
        remoteConfig.fetch { [weak self] status, _ in
            if status == .success {
                remoteConfig.activate(completion: nil)
            } else {
                print("Config not fetched")
            }
            
            let message = remoteConfig["message"].stringValue ?? ""
            let confirmAction = UIAlertAction(title: "확인하기", style: .default) { _ in
                //Google Analytics에서 이벤트 확인
            }
            
            let cancelAction  = UIAlertAction(title: "취소", style: .cancel, handler: nil)
            let alertController = UIAlertController(title: "깜짝 이벤트", message: message, preferredStyle: .alert)
            
            alertController.addAction(confirmAction)
            alertController.addAction(cancelAction)
            
            self?.present(alertController, animated: true, completion:nil)
            
        }
    }
}
```

해당 함수를 다음과 같이 구현한다.

이제 실제 이벤트를 기록하는 객체를 만들어야 하는데, 다행히 Analytics에서 이를 지원한다.

**import** FirebaseAnalytics

해주시고, 위에서 비워놓았던 confirmAction의 핸들러에서 

```swift
let confirmAction = UIAlertAction(title: "확인하기", style: .default) { _ in
                //Google Analytics에서 이벤트 확인
                Analytics.logEvent("promotion_alert", parameters: nil)
            }
```

Analytics의 logEvent에서 우리가 아까 만들었던 이벤트를 지정해주면, 확인하기 버튼이 눌릴 때마다 이벤트에 기록된다!

이제 버튼이 눌릴 때마다 정말로 이벤트가 잘 잡히는 지 확인해보자.

xcode 상단 메뉴 product > Scheme > Edit Scheme > Run > Arguments >A Arguments Passed on Launch에서 추가 (-FIRDebugEnabled)

해당 코드가 적용된 상태로 시뮬레이터를 실행시키는데

공지사항 팝업이 숨겨져야만 showEventAlert()이 보이기 때문에 두 가지 설정을 해준다.

Firebase Remote Config에서 점검이 끝났다고 가정하고 isHidden 매개변수를 true로 바꿔준다. 그 다음 이벤트가 찍히는지 실제로 보기 위해, 애널리틱스 섹션으로 이동해 DebugView라는 곳으로 이동한다.

시뮬레이션을 다시 실행시켜보면 Debug 기기의 시뮬레이션 액션이 그래프로 잡히는 것이 보인다.

![아날리틱스](https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/b7b4e283-21f5-4eff-970a-525e61776a15)

이벤트 버튼이 눌리는 것을 확인하는 것 까진 체크했다.

그러면 사용자의 반에게는 기본 메세지를 나머지 반에게는 실험값이 정말로 반반씩 잘 보여지는 지는 어떻게 확인할 수 있을까?

바로 AppDelegate에 코드를 입력해서 확인할 수 있다.

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        FirebaseApp.configure()
        Installations.installations().authTokenForcingRefresh(true) { result, error in
            if let error = error {
                print("ERROR")
                return
            }
            guard let result = result else {return}
            print("Installation auth token: \(result.authToken)")
            
        }
        return true
    }
```

초기화 코드 밑에 다음과 같은 코드를 추가하는데, 이 코드가 무엇을 의미하냐면 Firebase가 각각의 기기에 부여한 인증 토큰 값을 콘솔에서 확인할 수 있도록 한 것이다.

다시 빌드하여 Installation auth token 을 복사하고,

A/B Testing 콘솔로 이동하여 ‘실행 중’을 클릭한다

그럼 실제 실험 중인 우리 테스트가 나오는데, [테스트 기기 관리]로 들어간다.

여기에 FIS 인증 토큰을 넣는 칸이 나오는데, 여기에 Installation auth token을 복사한다.

여기서 비교값으로 선택한다음 저장하면 아까 시뮬레이터에서 나오지 않았던 반대 값이 alert에 나타나는 것을 알 수 있다.

<img width="563" alt="스크린샷 2023-12-26 오후 8 15 58" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/5f347b54-0bb3-426d-bf7e-8abf2122ebe9">
<img width="508" alt="스크린샷 2023-12-26 오후 8 18 52" src="https://github.com/jinyongyun/Realtime_Notice_Alert_APP/assets/102133961/42d856df-ef49-4b5d-ac10-cf89728fdc7f">














