# 3. 웹 API 기반 람다 함수

## 3.1 Amazon API Gateway 소개

> **Lambda에 대해 공부하는데 API Gateway에 대해 소개하는 이유**
> 

⇒ API Gateway를 사용하면 웹 API를 Lambda로 구현한 백엔드 기능에 매핑 할 수 있다! 

### **API Gateway 특징**

- 여러 스테이지(개발, 테스트, 운영 등)를 가질 수 있는 API를 구축할 수 있다
- HTTP 메서드 ⇒ 람다 함수로 구현
    
    ![스크린샷 2023-09-14 오후 4.35.35.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/bd4d09f4-dd03-4dd2-89e0-66d916d22532/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4.35.35.png)
    
- 접속 도메인명이 자동 생성되며, 원하는 경우 정의 가능

## 3.2 API 생성하기

1. **API Gateway 콘솔로 들어간다**

![스크린샷 2023-09-14 오후 4.59.26.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/1ee3a4ca-9ef6-4013-a0f2-26ae73cc71dd/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4.59.26.png)

: REST API 의 Build 버튼 클릭

**2. API 이름과 설명을 쓴다**

![스크린샷 2023-09-14 오후 5.02.41.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/09fd548f-7c61-4382-98cf-8af8300c8861/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.02.41.png)

: New API > `Create API` 버튼 클릭

📍 API를 새로 생성하거나 Swagger에서 불러오거나 예제를 이용하는 방법이 있다 

**3.신규 리소스를 생성한다**

![스크린샷 2023-09-14 오후 5.07.54.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/b3dc730a-ccfd-4b86-8c66-bcab6ef3e676/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.07.54.png)

![스크린샷 2023-09-14 오후 5.09.39.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/4c491984-2021-425b-b7e7-f3dc5568158b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.09.39.png)

: Create resource 버튼 클릭 > 리소스명을 ‘greeting’으로 입력
📍  이때 책에 나온 것 처럼 Greeting으로 입력 시 대소문자를 구분해줌…😇 소문자로 다시 생성함!

4.**메서드 생성**

![스크린샷 2023-09-14 오후 5.12.45.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/9f796e48-bf6d-4213-86e2-291ad61e237d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.12.45.png)

: Create Method 버튼을 클릭

## 3.3 API 연동하기

1. 메서드 생성

![스크린샷 2023-09-14 오후 6.08.54.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/b14302e7-61de-4118-90fb-871169ddd1a3/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6.08.54.png)

: Method Type을 `Get`으로 선택 >  API Gateway는 다양한 연동 방식(Integration Type)을 지원하는데 우린 Lambda 선택
> 이전 챕터에서 만든 람다 함수 선택 > Create Method 클릭!

📍 빠른 연동을 위해 두 개의 서비스를 동일 리젼에 해두자! 

1. **메서드 생성 후 API Gateway 홈으로 간다!**

![스크린샷 2023-09-14 오후 6.14.43.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/97594b98-21a1-45a4-b09d-e7e1c89d4fe2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6.14.43.png)

📍 실습 화면과 달리 API Gateway의 UI가 살짝 달라졌습니다 Test 탭으로 변경되었습니다 🎉

1. **매개변수 설정하기** 

![스크린샷 2023-09-14 오후 6.23.26.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/a6fa52e3-67d5-44ad-acb7-a149f957486c/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6.23.26.png)

![스크린샷 2023-09-14 오후 6.23.00.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/714e67d0-ee0e-4fd4-9ad3-905d8268b128/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6.23.00.png)

Method Request 탭의 `edit` 클릭 → 2번 화면에서 URL Query String Parameters 섹션에 name 추가! 

1. **매핑 템플릿 추가하기** 
    
    ![스크린샷 2023-09-14 오후 6.28.03.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/66ddb4e9-9e03-4445-b7cc-432a510ea2d7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6.28.03.png)
    
    : Integration Request 탭 > Mapping Templates > Create Template 클릭!
    
    ![스크린샷 2023-09-14 오후 6.27.22.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/3403cce9-f375-4823-8232-a398d684fde0/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6.27.22.png)
    
    : 매핑 탬플릿을 생성한 후 Create Template 클릭! 
    `{"name" : "$input.params('name')"}` :  name의 입력 매개변수 값을 반환한다
    

## 3.4 & 3.5 연동 테스트하기, 응답 변환하기

1. **테스트하기**

![스크린샷 2023-09-14 오후 7.47.37.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/ab6cbd2e-987f-456b-9c77-30a88291cc7a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7.47.37.png)

: Test  탭으로 가서 쿼리 스트링에 name 입력값을 입력 후 test버튼 클릭!
배

1. **배포하기**

![스크린샷 2023-09-14 오후 7.44.35.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/27e8ed0a-2baf-49a8-a005-7a0698f8aca0/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7.44.35.png)

: deploy API를 누른 후 > new stage → prod 입력 후 배포한다! 🎉🎉

### API 게이트웨이 내 옵션

- 결과 캐싱
- 스로틀링
- Amazon CloudWatch를 통한 통계 분석 및 로그 처리
- 자동 SDK 생성

등이 있다!

### 배포 기록 확인하기

![스크린샷 2023-09-14 오후 7.53.23.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/9366580c-0b00-4ec7-b0cb-0ee1f6d8dd21/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7.53.23.png)

: stage 탭에 들어가 Deployment history 탭을 누르면 지금까지의 배포 기록을 확인 할 수 있다.
📍 ▼ 최근 배포 시 오류가 있으면 쉽게 롤백도 가능하다 + 다른 스테이지로 같은 내용을 바로 배포도 가능함

![스크린샷 2023-09-14 오후 8.06.18.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/1f72258e-d158-4595-a824-c7a972f7e3a6/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.06.18.png)

## 3.6 매개변수로 리소스 경로 사용하기

![스크린샷 2023-09-14 오후 8.12.21.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/45696c07-cb26-4138-82e0-835e8854aada/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.12.21.png)

⇒ 바로 위 실습처럼 동일하게 리소스를 생성하고 메소드를 생성해준다! 

- **{username}**을 **리소스 경로**로 해서 새로운 사용자 리소스를 만든다
    - API 게이트웨이에 의해 **매개변수로 해석**
- 매핑 템플릿을 만들 때 `$input.params(’username’)`으로 키를 매핑해준다!

![스크린샷 2023-09-14 오후 8.19.11.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/a7cba307-7232-4db6-84ef-cb119454b5bd/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.19.11.png)

⇒ 배포 시 이미 username을 매개변수로 인식해서 자동으로 가져온다!👍🏻

## 3.7 API 게이트웨이 콘텍스트 사용하기

> 람다함수를 API 게이트웨이가 제공하는 콘텍스트에 통합해보자!
> 

<aside>
💡 **실습**
`$context.identity.sourceIp` 변수를 이용해 공용 IP주소를 반환하는 웹 API를 구축해보자!

</aside>

**개발 서비스 흐름** 

- `$context.identity.sourceIp` 값을 API Gateway에서 백엔드 Lambda 함수에 전달한다
- 입력값을 가져와서 결과와 같은 값을 반환하는 기본 람다 함수를 구현한다!

1. **새로운 함수 만들기** 
    
    ![스크린샷 2023-09-14 오후 8.50.14.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/2b55ef7f-6abe-449c-87a8-ebaac9c38607/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.50.14.png)
    
    : 기존 chapter에서 배운대로 함수명이 whatIsMyIp인 람다 함수를 만든다
    
2. **API Gateway 에 리소스 생성 후 메서드 실행한다!** 
    
    ![스크린샷 2023-09-14 오후 9.02.12.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c83d9de0-c055-40be-92e6-a905a5c039c7/12633184-9bcc-41b7-8ec0-330de3d7fa43/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.02.12.png)
    
    : 나의 IP가 출력된다! 🎉🎉
    

### 웹 API 서버리스 구현의 장점

- 가용성과 확장성이 AWS에 의해 관리됨
    - 관리에 대한 부담이 적고, 호출량에 따라서만 과금
- 매월 100만 회 이하의 호출에 대해서는 별도 비용은 발생하지 않는다

### 📍정리

- API Gateway 사용하면, 람다 함수로 만든 API에 대해서 외부에서도 접근이 가능하고 JSON 문법으로 반환하도록 설정 할 수 있다!
