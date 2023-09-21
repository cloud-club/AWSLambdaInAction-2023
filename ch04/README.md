# - **교재와 현재의 차이점 및 개정 내용 :** [IAM](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/introduction.html) DashBoard (2023.09.21 업데이트)

### **✅  위치 이동 및 뷰 추가**

- 보안 향상을 위한 조언 파트가 상단으로 이동
- Sign-in URL for IAM users가 좌측으로 이동하며, AccountID 및 Alias를 포함한 Account 확인용 뷰 추가
- IAM Resources 표기 방식의 변경
- Quick links 뷰가 추가 (2021년 9월)

<br>

### **✅  기능의 개정(Update)**

- `Credential Report` 만 제공되었으나, 상위 카테고리로 `Access reports` 생성되며 Access analyzer, Organization activity, Service control policies(SCPs) 항목이 추가되었다.
    - [Access analyzer](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/what-is-access-analyzer.html) : AWS 계정의 보안 및 권한 관리를 개선하고 AWS 리소스에 대한 액세스 권한을 더욱 효과적으로 분석하고 관리하는 데 사용되는 기능 (2019년 12월 도입)
    - [Organization activity](https://docs.aws.amazon.com/ko_kr/organizations/latest/userguide/orgs_introduction.html) : AWS Organization 내에서 발생하는 다양한 활동 및 이벤트를 모니터링하고 기록하는 도구 (2016년 6월 도입)
    - [Service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) : AWS Organizations을 사용하여 AWS 계정 그룹을 관리하고 권한을 제어하기 위한 도구 (2016년 6월 도입)
- `Encryption Keys`는 AWS에서 `AWS Key Management Service (KMS)`로, 별도로 분리되었다. (2017년 5월)

<br>

_**🫧Before**_

<img src="https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E3%85%87%E3%84%B9.png" width=800/>
<br>

_**🫧After**_

<br>
<img src="https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202023-09-21%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.34.28.png" width=800/>

<br>
<br>

# - 교재 요약 정리
💡 **IAM 사용자, 그룹과 역할을 통해 인증키를 사용한 클라우드 자원 보안 향상 방법** 

<img width="419" alt="스크린샷 2023-09-21 오후 4 09 02" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/be8335b3-9353-4f4e-8b7c-1837becc72b5">


IAM (Identity and Access Management): AWS 서비스에 접근하는 보안 주체(Identity)와 인증, 인증된 보안 주체의 서비스 접근(access)권한을 관리하는 서비스

비용적인 측면에서 무료이거나 (IAM) 비용이 거의 발생하지 않기 때문에 Amazon Cognito를 사용하는 것이 좋다.

## 4.1 IAM 사용자, 그룹과 역할

AWS의 모든 통신과정에서 **누가 API를 호출하는지(인증 부분), 필요한 부분을 가지고 호출을 하는지(권한 부분)**을 확인한다.

일반적으로 루트계정은 처음 로그인 시에만 사용하고 일반적으로는 IAM 사용자를 생성하여 제한된 권한을 사용하는 것을 권장 

cf. MFA(Multi-Factor Authentication) 스마트폰에 있는 MFA를 이용해 2중보안을 권고

<img width="427" alt="스크린샷 2023-09-21 오후 4 21 42" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/b410a169-1efa-41ed-bc05-377890df4e98">


<img width="424" alt="스크린샷 2023-09-21 오후 4 27 27" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/c33f31db-d49f-434d-85c8-a3dceef461b8">


사용자 그룹을 만들어 IAM 계정 배분 가능 

IAM 그룹과 사용자와 함께 역할 생성이 가능한데 **그룹과 역할의 가장 큰 차이점**은 

1. IAM 사용자는 그룹에 속할 수 있고, 그룹에 있는 권한을 상속받는다.
2. IAM 사용자, 애플리케이션,AWS 서비스는 각각 IAM 역할을 부여받을 수 있고 이때 IAM 역할에 설정된 권한을 상속받음

<img width="427" alt="스크린샷 2023-09-21 오후 4 32 02" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/349943dc-3dd5-4824-ac17-74c429148d13">


IAM 사용자,그룹,역할에는 한개 이상의 IAM 정책을 사용할 필요가 있는데 IAM 정책은 AWS 서비스의 실제 권한을 부여하는 것으로 IAM 사용자, 그룹 혹은 역할에 AWS 자원에 대한 허가 여부를 결정

또한 IAM 정책을 사용하면 필요한 권한 부분을 IAM 사용자, 그룹과 역할에 부여 할 수있다. 

<img width="425" alt="스크린샷 2023-09-21 오후 4 37 47" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/6673a62f-1b8a-41ea-90ba-0cf54e68d9fe">


인증(authentification)부분을 사용하려면 보안 인증키(security credential)가 필요한데, 루트 계정은 보안상 가급적 사용을 지양하고 IAM 사용자는 영구적인 보안키를 가질 수 있다. IAM 역할은 보안키를 가질 수 없으나 사용자나 특정 애플리케이션 및 AWS 서비스가 역할을 부여받으면 임시 보안키를 발급하여 일시적으로 IAM에 접근이 가능, 임시보안키는 유효기간이 지나면 AWS CLI와 SDK에 의해 자동으로 재발급된다.

**IAM 사용자(및 루트 계정) 의 보안 인증키 구성**

- 1개의 Access key ID : 임시 아이디 역할
- 1개의 Secret access key : 임시 암호 역할

**임시 보안 인증키 구성** 

- 1개의 Access key ID
- 1개의 Secret access key
- 1개의 보안(혹은 세션)토큰

**요약 및 정리** 
루트 사용자는 AWS 계정의 모든 권한을 소유하기 때문에 IAM 그룹과 IAM 사용자를 만들어 필요한 권한만 선별,부여하여 보안을 강화해야한다. 

---

## 4.2 IAM 정책 이해하기

**💡 IAM 정책을 이용해 AWS 자원에 권한을 주는 방법**

<img width="439" alt="스크린샷 2023-09-21 오후 4 46 36" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/69b43f4a-23b1-4ad0-ab92-6baab160a279">


IAM 정책은 특정 자원 접근에 대한 허가/거부를 하도록 AWS 서비스와 API 호출을 지정하여 접근을 허용하도록 함 

보안상의 이유때문에 모든 권한을 주는 것 보다 필요한 AWS 서비스와 API 호출을 모두 나열하여 IAM 정책을 만드는 것을 권장

<img width="416" alt="스크린샷 2023-09-21 오후 4 49 46" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/59eea13f-a20d-4905-889c-ce904d889799">


**IAM 정책의 타입** 

- **IAM 사용자-기반 정책** : 사용자, 그룹, 역할에 부여할 수있으며 사용자가 무엇을 할 수 있는지 설명
- **AWS 자원-기반 정책** : AWS 자원에 직접 부여할 수 있으며 누가 이 자원에 대해 접근이 가능한지 설명
- **신뢰-기반 정책** : 누가 IAM 역할을 맡을 수 있는지 설명,AWS Lambda가 사용하는 역할은 특정 신뢰-기반 정책을 사용하여 함수들이 역할을 사용할 수 있도록 한다.

<img width="428" alt="스크린샷 2023-09-21 오후 5 00 30" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/8eff8d78-18c5-44d8-bc3f-8d738bef204f">


IAM 정책은 사용자, 그룹 혹은 역할에 직접 지정할 수 있고 IAM 정책관리에서 쉽게 관리할 수 있다. 

## 4.3 IAM 정책 사용하기

**💡 IAM 문법을 이용해 Amazon S3에 접근을 허용하는 권한 부여에 필요한 기능 설명** 

<img width="418" alt="스크린샷 2023-09-21 오후 5 08 34" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/cefe15bf-21c8-4523-ac60-60d85ecb290f">


Amazon S3버킷에 읽기/쓰기 접근 정책 

<img width="327" alt="스크린샷 2023-09-21 오후 5 10 31" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/14f5f1d1-c901-4db9-91ae-d987fd40c2cb">


Amazon S3 웹 콘솔을 이용해 필요한 권한 주기

<img width="403" alt="스크린샷 2023-09-21 오후 6 31 32" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/8e1ef4b4-636b-4ead-bbe4-a461a3e31d0a">


DynamoDB 테이블에 읽기/쓰기 권한을 주는 정책 

**요약 및 정리** 

IAM 문법을 이용하여 정책에 다양한 권한을 부여하는 방법을 배웠고 IAM정책의 보안 특성상 필요하지않은 권한을 처음부터 많이 주면 보안상 문제가 발생할 수 있고 권한을 향후에 제거할 때 애플리케이션 구동에 문제를 일으킬 수도 있기 때문에 필요한 최소한의 권한을 부여하는 것을 원칙으로 해야한다 (cf. 최소한의 권한 부여 원칙) 

> “모든 프로그램과 권한을 가진 사용자는 최소한의 권한을 사용하여 필요한 기능을 수행해야 한다”
[관련 논문](https://dl.acm.org/doi/10.1145/361011.361067)
> 

---

## 4.4 IAM 정책 변수 사용하기

정책 변수(Policy variable)란 ? 
정책 내에서 동적 값을 정의하여 IAM 정책 내에서 사용하는 것 

<img width="433" alt="스크린샷 2023-09-21 오후 6 40 37" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/87bc15ed-fa8c-4451-aa1d-2e01a47a41dd">


<img width="422" alt="스크린샷 2023-09-21 오후 6 40 46" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/64996121/6f996757-55c6-465f-9057-aff896df2359">


cf. [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html# policy-vars-infotouse](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html#%20policy-vars-infotouse).

## 4.5 IAM 역할 사용하기

IAM 역할을 사용하는 것은 임시 보안 인증키를 사용하여 람다함수 내의 코드가 다른 AWS자원에 대해 접근하여 작업할 수 있다는 것을 뜻함

IAM 역할 사용 시 임시 보안 인증키가 자동으로 교체 → AWS 보안 인증키를 넣을 필요가 X 

IAM 역할 장점 

- 인증키가 외부로 유출되는 위험을 피할 수 있음
- Github 또는 Bitbucket의 공용 코드 저장소에 코드를 커밋할 때 실수로 외부에 공개될 가능성을 없애줌

IAM 역할이 사용 할 수 있는 정책 

- 사용자-기반의 정책은 역할이 부여하는 권한을 설명
- 신뢰-기반 정책은 역할이 할 수 있는 것으로 예를들어 IAM 사용자에 따라 사용할 수 있는 AWS 자원 사용방법

→ 항상 AWS 콘솔에서 생성한 역할을 찾아서 무엇이 허용되는지 여부를 파악하는 것이 필요
