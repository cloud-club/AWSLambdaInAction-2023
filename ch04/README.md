## - **교재와 현재의 차이점 및 개정 내용 :** [IAM](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/introduction.html) DashBoard (2023.09.21 업데이트)

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

## - 교재 요약 정리
