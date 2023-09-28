이 장에서 살펴볼 내용

- 예시 인증 서비스를 위한 서버리스 아키텍처 구현하기
- 람다 함수를 활용하여 백엔드 생성
- 클라이언트 애플리케이션 구현을 위한 브라우저에서 구동되는 자바스크립트와 HTML 이용하기
- AWS CLI를 활용한 초기화 자동화 및 배치
- 복수의 람다 함수 간 코드 공유 및 통합 구성
- 관리할 서버가 없는 상태에서 Amazon SES를 활용하여 이메일 보내기

# 업데이트 해야 하는 부분

최신 node.js 코드로 업데이트는 못 함

1. init.sh
    1. 더 이상 지원하지 않는 `nodejs4.3` 대신, → `nodejs16` 으로 바꿔줘야 함
2. deploy.sh
    1. S3 정책이 바뀌어서 그런지(정확하지 않음) `-acl public-read` 를 붙이면 안됨
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/6636b06f-5784-4cb4-a075-824c12afb92a)
    
    이후 S3 버킷을 공개해야 하므로 다음과 같은 절차를 따라야 함
    
    1. Amazon S3 → 버킷 → <버킷 이름> → 권한 → 편집 → 모든 퍼블릭 액세스 차단 해제 → 변경 사항 저장 → 버킷 정책 편집→ 정책 json 추가 → 저장
        
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "PublicReadGetObject",
                    "Effect": "Allow",
                    "Principal": "*",
                    "Action": "s3:GetObject",
                    "Resource": "arn:aws:s3:::<버킷 이름>/*"
                }
            ]
        }
        ```
        
    2. 이후 원하는 객체에 url로 접근
        
        ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/46004dbb-4b67-4f8f-8cd9-4bb7e841303c)
        
3. SES에 본인 이메일 자격 증명 생성 해야함
    
    Amazon SES → 자격 증명 생성 → 이메일 주소 → 본인거 입력 → 자격 증명 생성 → 이메일 인증
    

# 9.3 코드 공유하기

- DRY 원칙을 지키기 위해 Lambda에 올라갈 코드를 패키징 해야함
- 비밀번호를 해싱하기 위한 `computeHash` 함수를 lib 폴더에 만들고 재사용

# 9.4 홈페이지 만들기

- index.html을 만들자
    - 저장소에 있는 코드는 부트스트랩을 사용함
        - `<a class="btn btn-info btn-lg" href="signUp.html">Sign Up</a>`
        - `btn` 이라는 class에 해당하는 css가 미리 준비돼있어서, 편하게 예쁜 css를 먹임

# 9.5 신규 사용자 등록하기

# 9.6 사용자 이메일 인증하기

요약

이번 장에서는 웹 클라이언트 애플리케이션을 사용하여 예시 인증 서비스를 만들었다. 특히 다음 사항에 대해 학습했다. 

- 백엔드를 구현하기 위해 여러 개의 람다 함수 사용
- 자바스크립트와 함께 HTML을 사용하여 웹 클라이언트 애플리케이션을 구현
- AWS CLI를 사용하여 자원 생성이나 AWS 업데이트를 자동화하기
- 애플리케이션에서 통합 구성이나 공유 코드 관리하기
- Amazon SES를 사용하여 이메일 전달하기
