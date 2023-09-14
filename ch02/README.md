## - 교재와 현재 차이점 (2023.09.13 업데이트)

- 교재에서는 함수 생성시 이름, 런타임. 역할을 설정할 수 있었고, 역할에서 사용자 지정 역할 생성, 기존 역할 선택, 템플릿에서 새역할 생성, 사용자 지정 역할 생성 등이 있었지만 ,현재 람다에서는 역할은 '기본 실행 역할 변경' 에서 선택할 수 있고, 아키텍처와 권한 항목이 생김.
    
    예시로 올려준 AWS console 화면 레이이웃이 달라져서 실습을 하면서 캡쳐 진행
    
- ⚠️ 교재의 2.2에서 함수를 입력 후 2.4에서 테스트를 하면 이전함수가 계속 출력
    - Deploy 버튼을 누르고 함수를 실행하면 해결 ( 새 코드 추가시 Deploy 누르고 실행 )
    - 교재에서 알려준 핸들러 필드 형식이 바뀌어서 함수가 실행이 안 된다고 예상
        
        ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/df40aea9-83e8-4080-8ac7-c7454b66321a)


        
- Python
    - 제공해준 Python 코드는 문제 없음
- Node.js
    - 제공해준 Node 코드는 문제 없음
    - ⚠️ index.js 함수 돌리는 과정에서 오류 발생
        - `greetingsOnDemand` 함수 제작하고 자동으로 생성되는 `index.mjs`의 확장자를 `.js`로 변경후 실행

## - 교재 요약 정리

### 2.0살펴볼 내용

- AWS Lambda 함수 만들어 보기
- 람다 함수의 구성과 설정 이해
- 웹 콘솔로 람다 함수 테스트해 보기
- AWS-CLI를 통해 함수 호출해 보기

### 2.1 새로운 함수 만들기

- 함수 생성하기
    
    > → `greetingsOnDemand` 함수 생성, `lambda_basic_excution` 역할 생성
    > 
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/fe3cc5b3-a1a5-45a4-a650-387246a45557)
    > 

- 람다 함수를 실행할 수 있는 트리거(Trigger) 선택
    
    > → 트리거 : 이벤트가 발생하는 각 AWS 서비스 지점으로 개별 서비스 이벤트에 따라 함수를 실행
    > 
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/07f20a6f-05d5-4ea4-aa1f-5315564e71d4)
    > 
    > - `ex)` 트리거에서 API Gateway를 선택할 경우
    >     
    >     → 웹 API가 람다 함수 혹은 AWS IoT를 호출하여 AWS IoT 플랫폼의 서버리스 백엔드 구성
    >     
    > 
    > - 함수에 코드를 넣는 방법
    >     - 웹 브라우저에서 직접 코드 편집
    >     - 로컬 환경에서 zip 파일 업로드
    >     - Amazon S3에서 zip 파일 업로드 ( CI 과정 가능 )

### 2.2 함수 작성하기

- `greetingsOnDemand` 함수
    
    > → Node.js
    > 
    > 
    > ```jsx
    > // 초기화
    > console.log('Loading function');
    > 
    > // 함수 선언 ( 입력 이벤트는 자바스크립트 객체 )
    > exports.handler = (event, context, callback) => {
    >     console.log('Received event:',
    >         JSON.stringify(event, null, 2));
    >     console.log('greet =', event.greet);
    >     console.log('name =', event.name);
    >     var greet = '';
    >     if ('greet' in event) {
    >         greet = event.greet;
    >     } else {
    >         greet = "Hello";
    >     }
    >     var name = '';
    >     if ('name' in event) {
    >         name = event.name;
    >     } else {
    >         name = "World";
    >     }
    >     var greetings = greet + ' ' + name + '!';
    >     console.log(greetings); // CloudWatch 로그
    >     callback(null, greetings); // 함수 종류 및 결과 반환
    > };
    > ```
    > 
    > - 함수 초기화
    >     
    >     → Lambda는 함수의 여러 호출에 대해 동일한 컨테이너 재사용 가능
    >     
    >     → 함수 호출될 때마다 초기화 X
    >     
    >     → 초기화 단계에서는 한 번만 실행될 수 있는 코드를 넣어야 함 (DB 연결, 캐시 초기화, 데이터 로드)
    >     
    > - CloudWatch Logs
    >     
    >     → Lambda는 디스플레이가 없는 헤드리스 환경에서 함수 실행하여 CloudWatch Logs에 로그 쌓임
    >     
    >     → Node.js의 경우 `console.log()` 에 대한 항목이 넘어감
    >     
    > - `greetingsOnDemand` 함수의 로직
    >     
    >     → 입력 이벤트에 `name 키`가 있으면 이름 사용, 없으면 기본 Hello World 출력
    >     
    > - 함수 종료
    >     
    >     → Node.js의 경우 callback을 사용하여 종료
    >     
    > - 입력의 `context`
    >     
    >     → 람다 함수 구성 및 실행을 처리하는 방법에 대한 정보 포함 (실행 시간 제한에 도달하기까지 남은 시간 확인)
    >     

### 2.3 다른 설정 지정하기

- 소스 코드를 작성하였다면 Lambda에서 코드 내에 어떤 함수를 호출해야 하는지 지정
    
    > → 핸드러 (Handler) 필드를 통해 지정 가능
    > 
    > - zip 파일의 경우 2개 이상의 파일이 존재 할 수 있으므로 `<file name>/<funnction name>`
    > - Node.js의 경우 index.js의 파일의 hander 함수 기본값은 `indes.handler`
    > - Node.js를 사용시 핸들러에서 사용하는 함수를 내보내기(export) 해야 함
    > - 소스 코드에서 다른 함수 이름을 사용하려면 핸들러에서 이름을 업데이트 해야 함
    > - 지정되지 않은 함수는 코드 내부적으로만 사용 가능
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/db92d961-3571-44d8-9eea-5c470d0555c8)
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/f841c2f0-71dc-4c1f-92b7-7dfb631ae85d)
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/4c7621bd-968f-46cb-a8ce-43952df81ac5)
    > 
    > AWS 내부 사설 네트워크안의 리소스 접근할 시 설정
    > 

### 2.4 함수 테스트하기

- 테스트 호출에 사용될 테스트 이벤트 준비
    
    > → 람다 함수 호출 시 이벤트는 실제 런타임에서 이벤트가 수신될 때 JSON 문법 사용
    > 
    > 
    > ```json
    > {
    >   "name": "chan"
    > }
    > ```
    > 

- 테스트 하기
    
    > → 테스트 이벤트 준비 후 테스트 실행
    > 
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/263784dd-183d-430c-8028-4137afd8f108)
    > 
    

### 2.5 Lambda API 함수 실행하기

→ 사실상 모든 함수는 AWS Lambda Invoke API 호출을 통해 실행 가능

![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/73b357c2-0b45-4138-aeb1-6677d4519c53)


- AWS CLI 설치
    
    > → IAM에서 **AmazonAPIGatewayAdministrator, AmazonCognitoPowerUser, AWSLambda_FullAccess** 로 사용자 생성 후 진행
    > 
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/2d7cbbf4-1d3c-4051-8f21-e1ed0e586d5b)
    > 
    
- AWS Lambda Invoke API 함수를 직접 호출
    
    > → AWS CLI 설치하여 테스트
    > 
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/01a8c1cb-259c-4c7d-a7ec-ca4fe29c1cfd)
    > 
    > ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/85063965/d83db937-edbc-43f6-8cdc-f6e1e6312f0c)
