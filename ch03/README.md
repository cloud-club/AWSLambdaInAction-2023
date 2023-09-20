# 3. 웹 API 기반 람다 함수

## 3.1 Amazon API Gateway 소개

> **Lambda에 대해 공부하는데 API Gateway에 대해 소개하는 이유**
> 

⇒ API Gateway를 사용하면 웹 API를 Lambda로 구현한 백엔드 기능에 매핑 할 수 있다! 

### **API Gateway 특징**

- 여러 스테이지(개발, 테스트, 운영 등)를 가질 수 있는 API를 구축할 수 있다
- HTTP 메서드 ⇒ 람다 함수로 구현
<img width="504" alt="스크린샷 2023-09-14 오후 4 35 35" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/15b850e1-b214-44fe-a6e9-385f42f5bb79">
    
- 접속 도메인명이 자동 생성되며, 원하는 경우 정의 가능

## 3.2 API 생성하기

**1. API Gateway 콘솔로 들어간다**
   
<img width="1462" alt="스크린샷 2023-09-14 오후 4 59 26" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/cd1e0de8-c549-42d9-b38a-fec711f301b2">

: REST API 의 Build 버튼 클릭


**2. API 이름과 설명을 쓴다**

<img width="717" alt="스크린샷 2023-09-14 오후 5 02 41" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/b79acaee-7faf-4aa3-b7bf-7ab1adcd834d">


: New API > `Create API` 버튼 클릭

📍 API를 새로 생성하거나 Swagger에서 불러오거나 예제를 이용하는 방법이 있다 


**3.신규 리소스를 생성한다**

<img width="847" alt="스크린샷 2023-09-14 오후 5 07 54" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/c7517a44-7250-43d8-ace2-05f286a3317e">

<img width="847" alt="스크린샷 2023-09-14 오후 5 09 39" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/d25374b7-d2a4-4586-9ac9-b51f82ff275c">

: Create resource 버튼 클릭 > 리소스명을 ‘greeting’으로 입력
📍  이때 책에 나온 것 처럼 Greeting으로 입력 시 대소문자를 구분해줌…😇 소문자로 다시 생성함!


**4.메서드 생성**

<img width="847" alt="스크린샷 2023-09-14 오후 5 12 45" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/5808da77-f263-4733-8b4c-1f9b3ffa29fc">


: Create Method 버튼을 클릭


## 3.3 API 연동하기

**4.메서드 생성**


<img width="847" alt="스크린샷 2023-09-14 오후 6 08 54" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/1fa3e358-148e-45b8-93b1-5886a1806f70">


: Method Type을 `Get`으로 선택 >  API Gateway는 다양한 연동 방식(Integration Type)을 지원하는데 우린 Lambda 선택
> 이전 챕터에서 만든 람다 함수 선택 > Create Method 클릭!

📍 빠른 연동을 위해 두 개의 서비스를 동일 리젼에 해두자! 

**5.메서드 생성 후 API Gateway 홈으로 간다!**

<img width="1469" alt="스크린샷 2023-09-14 오후 6 14 43" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/11373fed-4530-4d9e-b66b-592686fa5e12">


📍 실습 화면과 달리 API Gateway의 UI가 살짝 달라졌습니다 Test 탭으로 변경되었습니다 🎉

**6. 매개변수 설정하기** 

<img width="1469" alt="스크린샷 2023-09-14 오후 6 23 26" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/5725b889-d10d-48ef-809e-1370591fdf54">


<img width="1469" alt="스크린샷 2023-09-14 오후 6 23 00" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/55ae868e-6599-4095-bcbc-c6bfbd34d9b3">

Method Request 탭의 `edit` 클릭 → 2번 화면에서 URL Query String Parameters 섹션에 name 추가! 

**7. 매핑 템플릿 추가하기** 
    
 <img width="1469" alt="스크린샷 2023-09-14 오후 6 28 03" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/7b2eba8e-1203-4bac-b278-d19f2b03b253">
    : Integration Request 탭 > Mapping Templates > Create Template 클릭!
    
<img width="1469" alt="스크린샷 2023-09-14 오후 6 27 22" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/be71a9c9-cbbc-45d7-86e8-e9f23c77c0f2">
    : 매핑 탬플릿을 생성한 후 Create Template 클릭! 
    `{"name" : "$input.params('name')"}` :  name의 입력 매개변수 값을 반환한다
    

## 3.4 & 3.5 연동 테스트하기, 응답 변환하기

**8. 테스트하기**

![스크린샷 2023-09-20 203159](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/108706692/25a000c3-a905-4b4e-a43f-b3e94ee75307)



: Test  탭으로 가서 쿼리 스트링에 name 입력값을 입력 후 test버튼 클릭!
배

**9. 배포하기**

<img width="1469" alt="스크린샷 2023-09-14 오후 7 44 35" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/46c00584-5380-413e-b2b2-5d3d1f0c177d">
: deploy API를 누른 후 > new stage → prod 입력 후 배포한다! 🎉🎉

### API 게이트웨이 내 옵션

- 결과 캐싱
- 스로틀링
- Amazon CloudWatch를 통한 통계 분석 및 로그 처리
- 자동 SDK 생성

등이 있다!

### 배포 기록 확인하기

<img width="1469" alt="스크린샷 2023-09-14 오후 7 53 23" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/7de823b7-50f7-4423-9a26-f43c279cf60f">


: stage 탭에 들어가 Deployment history 탭을 누르면 지금까지의 배포 기록을 확인 할 수 있다.
📍 ▼ 최근 배포 시 오류가 있으면 쉽게 롤백도 가능하다 + 다른 스테이지로 같은 내용을 바로 배포도 가능함

<img width="1469" alt="스크린샷 2023-09-14 오후 8 06 18" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/0bfd1a5a-39a3-47fb-bb2b-cb96620f68c9">


## 3.6 매개변수로 리소스 경로 사용하기

<img width="1469" alt="스크린샷 2023-09-14 오후 8 12 21" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/2752b55b-7542-436d-9722-22ee4de0eacc">


⇒ 바로 위 실습처럼 동일하게 리소스를 생성하고 메소드를 생성해준다! 

- **{username}**을 **리소스 경로**로 해서 새로운 사용자 리소스를 만든다
    - API 게이트웨이에 의해 **매개변수로 해석**
- 매핑 템플릿을 만들 때 `$input.params(’username’)`으로 키를 매핑해준다!

<img width="1244" alt="스크린샷 2023-09-14 오후 8 19 11" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/58dd8645-d8e7-4e59-9c45-a9c972690602">


⇒ 배포 시 이미 username을 매개변수로 인식해서 자동으로 가져온다!👍🏻

## 3.7 API 게이트웨이 콘텍스트 사용하기

> 람다함수를 API 게이트웨이가 제공하는 콘텍스트에 통합해보자!
> 

<aside>
💡 실습
`$context.identity.sourceIp` 변수를 이용해 공용 IP주소를 반환하는 웹 API를 구축해보자!

</aside>

**개발 서비스 흐름** 

- `$context.identity.sourceIp` 값을 API Gateway에서 백엔드 Lambda 함수에 전달한다
- 입력값을 가져와서 결과와 같은 값을 반환하는 기본 람다 함수를 구현한다!

**1. 새로운 함수 만들기** 
    
<img width="1470" alt="스크린샷 2023-09-14 오후 8 50 14" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/c31e2b38-88cb-4da3-8e94-de811c6e7dce">

: 기존 chapter에서 배운대로 함수명이 whatIsMyIp인 람다 함수를 만든다
    
**2. API Gateway 에 리소스 생성 후 메서드 실행한다!** 
    
<img width="1470" alt="스크린샷 2023-09-14 오후 9 02 12" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/66239292/d5b90ac3-6c18-4081-b7ac-baba63ac155f">

: 나의 IP가 출력된다! 🎉🎉
    

### 웹 API 서버리스 구현의 장점

- 가용성과 확장성이 AWS에 의해 관리됨
    - 관리에 대한 부담이 적고, 호출량에 따라서만 과금
- 매월 100만 회 이하의 호출에 대해서는 별도 비용은 발생하지 않는다

### 📍정리

- API Gateway 사용하면, 람다 함수로 만든 API에 대해서 외부에서도 접근이 가능하고 JSON 문법으로 반환하도록 설정 할 수 있다!
