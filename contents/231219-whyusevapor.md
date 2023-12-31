# 🤿 HTTP Basics: Why use Vapor?

Swift와 Vapor를 사용한 서버 측 앱 개발은 특별한 경험입니다.<br>
기존의 많은 서버 측 언어(예: PHP, JavaScript, Ruby)와 달리 Swift는 강력하고 정적으로 유형화되어 있습니다.<br>
이러한 특성 덕분에 iOS 앱의 런타임 충돌 횟수가 크게 감소했으며, 서버 측 앱에서도 이러한 이점을 누릴 수 있습니다.<br>

서버 측 Swift의 또 다른 잠재적 이점은 성능 향상입니다.<br>
스위프트는 컴파일된 언어이기 때문에 스위프트를 사용하여 작성된 앱은 해석된 언어로 작성된 앱보다 성능이 더 우수할 가능성이 높습니다.<br>

하지만 서버 측 Swift 앱을 작성해야 하는 가장 큰 이유는 Swift를 사용할 수 있기 때문입니다.<br>
Swift는 가장 빠르게 성장하고 가장 사랑받는 언어 중 하나로, 여러 언어의 장점을 결합한 현대적인 구문과 기능을 갖추고 있습니다.<br>
현재 iOS용으로 개발 중이라면 이미 이 언어를 잘 알고 있을 것입니다.<br>
즉, 서버 측 앱과 iOS 앱 간에 핵심 비즈니스 로직 코드와 모델을 공유할 수 있습니다<br>

Swift를 선택하면 서버 애플리케이션을 개발할 때 Xcode를 사용할 수 있습니다.<br>
리눅스의 파운데이션은 iOS와 macOS에서 볼 수 있는 기능의 일부이지만, 대부분의 개발은 Xcode에서 할 수 있습니다.<br>
따라서 대부분의 서버 측 언어에는 없는 IDE의 강력한 디버깅 기능에 액세스할 수 있습니다.<br>

## 🤿 Vapor and the server-side Swift ecosystem.

Vapor는 주요 서버 측 Swift 프레임워크의 상위 레벨로 부상했으며, 개발자들은 Swift 서버 작업 그룹(Swift Server Work Group, SSWG)과 긴밀히 협력하고 있습니다.<br>

SSWG는 서버에서 Swift 사용을 촉진하는 운영 팀입니다.<br>

Vapor 코어 팀을 포함하여 Apple과 커뮤니티의 엔지니어로 구성되어 있습니다<br>

SSWG는 또하느 PostgreSQL 드라이버 및 메트릭 라이브러리와 같이 SwiftNIO를 기반으로 구축된 권장 프로젝트를 위한 인큐베이션 프로세스를 갖추고 있습니다.<br>

Vapor도 SwiftNIO를 기반으로 구축되었기 때문에 서버 측 Swift 앱과 함께 이러한 패키지를 사용할 수 있습니다.<br>

이러한 프로젝트 중 상당수는 이미 Vapor에 통합되어 있습니다.<br>
