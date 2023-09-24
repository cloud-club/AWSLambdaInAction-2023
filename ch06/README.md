# 사용자 관리하기

Amazon Cognito?

외부 사용자와 애플리케이션이 AWS에서 역할을 사용하여 임시 자격 증명을 사용할 수 있도록 특별히 설계된 서비스. AWS 보안 베스트 케이스들을 따라하기 쉽게 만들어놓음. Amazon Cognito가 AWS 자격 증명을 클라이언트에게 제공할 수 있다.

## 6.1 Amazon Cognito Identity

Amazon API Gateway를 통해 제공되는 API들에 대해서 제한 접근을 두어서 보호하고 싶다면 API Gateway 웹 콘솔에서 Request의 인증 모드를 AWS IAM 인증 타입으로 설정하면 된다.

<img width="700" alt="image" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/49530253/43a4dc4e-0832-4926-8ca8-e84546fadabb">


Cognito는 AWS 자격 증명뿐만 아니라 많은 것들을 제공. 외부에서 연동시켜서 사용자 인증 방법을 적용할 수도(페이스북, 트위터 같은)

외부 인증 ⇒ Cognito가 고유한 ID 값을 부여함 ⇒ 디바이스 상관 없이 사용자 인식할 수 있도록 함 ⇒ 인증되지 않은 사용자의 경우 디바이스에 ID를 고정시킴

즉, Cognito는 인증/미인증 두 가지 방법의 인증을 지원하게 된다. 인증의 경우 사용자별로 디바이스 상관 없이 Cognito가 부여하는 고유 ID로 사용자를 인식한다. 미인증의 경우 디바이스에 ID를 고정시켜서 인증 사용자와 구분한다.

한 AWS 계정은 여러 애플리케이션을 관리할 수 있다. Cognito는 의존성을 가지지 않는 독립된 여러 identity를 사용할 수 있도록 해준다. ⇒ Cognito에 있는 pool 중 어느 것을 이용할지 명시하여 특정 pool만 사용하도록 할 수 있다. 각 Identity pool은 IAM 인증 역할을 가진다. 미인증 사용자들을 Identity pool에 넣을 수 있다.

즉, Identity pool은 IAM 인증 역할을 가지고 하나의 AWS 계정은 독립적인 여러 Identity를 가질 수 있다. 또 미인증 사용자들을 Identity pool에 넣어서 미인증 역할을 사용할 수 있다.

[Cognito를 활용한 미인증 사용자 관리 흐름]

<img width="700" alt="image" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/49530253/096ff42f-bd27-41ec-ab5f-112a5295ab33">


1. 사용자는 클라이언트 애플리케이션 사용
2. Amazon Cognito에 Identity pool ID를 보냄
3. Identity pool이 미인증된 사용자도 허용되도록 설정되어 있다면, 클라이언트는 Identity ID를 받아서 디바이스에 저장 ⇒ 이 ID는 재사용 가능 
이때 자격증명도 가져올 수 있어서 Identity pool이 가진 미인증 역할도 받아올 수 있음
4. Amazon Cognito에서 받은 AWS 자격증명으로 다른 AWS 서비스 사용할 수 있는 접근 권한을 가짐. 이때 어디까지 접근 가능한지는 Identity pool에서 받은 미인증 역할에 부착된 정책 따름.

클라이언트 애플리케이션에서 초반에 필요한 유일한 정보는 identity pool ID

## 6.2 외부 아이덴티티 제공자

Amazon Cognito는 여러 외부 아이덴티티 제공자를 지원한다

- Amazon
- Facebook
- Twitter
- Digits
- Google

Cognito는 외부 아이덴티티에서 제공한 인증 토큰을 신뢰한다.

[Cognito와 사용자 인증 서비스를 활용한 전체적 흐름]

<img width="700" alt="image" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/49530253/80609172-2ce5-4577-8d7a-d58fd05105d1">

1. 사용자는 클라이언트 애플리케이션 사용
2. 클라이언트는 외부 아이덴티티를 사용해서 인증을 받음(AWS와 관계 없음)
3. 클라이언트 애플리케이션은 제공자로부터(google 등) 인증 토큰을 받음
4. Cognito는 토큰이 유효한지 확인
5. 클라이언트 애플리케이션은 Identity pool ID와 받은 토큰을 Cognito에 보내서 Identity ID를 받아옴
6. AWS 자격증명을 통해서 다양한 액션 및 자원에 접근 가능(미인증 흐름과 동일)

클라이언트 애플리케이션에서 초반에 필요한 유일한 정보는 identity pool ID

## 6.3 사용자 인증 통합하기

외부 아이덴티티 제공자들에 의한 인증 외에도 다양한 사용자 인증 사용 가능

인증 서비스는 Amazon Cognito에서 인증 토큰을 유효한지 확인하는 백엔드 호출이 있어야 한다. 

<img width="700" alt="image" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/49530253/2b10da66-60e3-4cce-b0fd-d2e25306f790">



1. 사용자는 클라이언트 애플리케이션 사용
2. 사용자 아이덴티티 제공자(외부 아이덴티티 제공자 이외 다른 사용자 인증) 를 통해서 인증받음
3. Amazon Cognito에서 인증 토큰 가져옴
4. 사용자는 Cognito에서부터 넘어온 토큰을 받아옴
5. Identity pool ID와 인증 토큰을 Cognito에 보내서 Identity ID를 가져옴. 이때 인증 토큰 유효한지 확인
6. AWS 자격증명으로 액션 및 자원 접근 가능

결국 사용자 인증 서비스에 의해 인증된 사용자나 외부 아이덴티티 제공자가 인증한 사용자는 동일한 능력을 가진다. AWS 서비스가 보기에는 똑같음.

## 6.4 인증 사용자와 미인증 사용자

Identity pool 사용법은 두가지가 있다.

1. 인증 사용자만 사용
2. 인증/미인증 사용자 모두 사용

이건 서비스에 따라서 다르다! 

Case1 ⇒ 인증은 안해도 미인증된 사용자 활동 내역을 분석할 때(인증/미인증 사용자 모두 사용)

Case2 ⇒ 서비스 시작할 때부터 인증을 받고 시작할 때(인증 사용자만 사용)

보통은 인증/미인증 사용자 모두 사용한다. 인증을 강제하는 것은 사용자 경험에 좋은 영향을 끼치지 않기 때문. 인증 없는 애플리케이션은 사용자별 맞춤형 서비스 제공이 어렵다는 단점도 있음.

## 6.5 Amazon Cognito 정책 요소 사용하기

실제 정책 요소들을 어떻게 사용하는지 알아보자
