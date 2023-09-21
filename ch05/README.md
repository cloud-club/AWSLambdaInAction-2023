# 5장

5장에서 살펴볼 내용

- 람다 함수에 사용자 라이브러리 적용하기
- 함수 실행을 위해 다른 AWS 서비스 활용하기
- S3 버킷, DynamoDB 테이블과 같은 백엔드 서비스 연동하기
- 바이너리 기반 함수 만들기
- 서버리스 얼굴 인식 함수 만들기
- 일정 주기로 함수 실행하기

# 5.3 함수에 바이너리 함께 사용하기

- 바이너리 혹은 의존성 있는 파일들이 OS의 공유 폴더에 기본값으로 되어 있기 때문에, 람다의 외부 함수는 복잡해질 수 있음.
- 예를 들어
    - Linux/Unix → /usr/lib or /usr/local/lib
    - Windows → DLLs
    - 에서는 배포 패키지에 쉽게 포함하고 제공할 수 있는 로컬 정적 빌드를 만들어야 함.
        - AWS Lambda가 사용하는 환경과 비슷해야 하므로, Amazon Linux EC2 인스턴스 사용을 권장

## 얼굴 인식 웹 앱을 만들어보자 - 실패!

### OpenCV 사용

- 이를 설치하면 OS의 공유 폴더에 기본값으로 설치됨
- 다른 곳에 설치하려면 OpenCV의 cmake와 같은 프로젝트의 설정 도구를 참고하여 옴션을 통해 정적 빌드 할 수 있음

### 함수 구현

1. 설계
    - input
        
        ```json
        {
        	"imageUrl": "http(s)://..."
        }
        ```
        
    - output
        
        ```json
        {
        	"faces": 3,
        	"outputUrl": "https://..."
        }
        ```
        
2. S3 버킷 만들기
    1. `aws s3 mb s3://mele-bucket`
    2. 버킷 정책
        
        ```json
        {
        	"Version": "2012-10-17",
        	"Statement": [
        		{
        			"Sid": "",
        			"Effect": "Allow"
        			"Pricipal": "*",
        			"Action": "s3:GetObject",
        			"Resource": "arn:aws:s3:::mele-bucket/*"
        		}
        	]
        }
        ```
        
3. 함수 작성(`/source` 폴더에 있음) - 아직 없음

### 함수 테스트하기

- `aws lambda invoke --function-name mele-bucket --payload '{"imageUrl":"{분석 할 이미지 주소}"}' --cli-binary-format raw-in-base64-out output.txt`
    - 이전 실습들과 마찬가지로 `--cli-binary-format raw-in-base64-out` 가 있어야 함.
- output.txt
    
    ```json
    {
    	"faces": 3,
    	"outputUrl": "https://mele-bucket.s3.amazonaws.com/tmp/<date + uniqueUUID>.jpg"
    }
    ```
    
- 이상적인 결과
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/14f81360-11c3-41b5-ba71-65d8bf6dbd03)
    

# 5.4 함수 스케줄링하기

- 지금까진 Amazon S3에서 발생하는 이벤트를 람다 함수에 구독시키는 방법을 배움
- Amazon EventBridge(구 Amazon CloudWatch Event)를 활용하면 람다 함수를 특정 AWS 플랫폼에서 발생하는 이벤트로 인해 실행되도록 할 수 있음
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/5cb0ece3-eee6-4612-a301-01c330a1d860)
    
    - Amazon EC2에서 관리하는 인스턴스의 상태 전이
    - 또는
    - 사용자가 정의한 스케줄에 따라 실행

## 반복 스케줄로 람다 함수 실행하기

- 이전에 만든 얼굴 마킹 이미지들을 1시간마다 정리하고 싶다면, 람다 함수를 사용하면 됨
    - 함수 작성(`/source` 폴더에 있음) - 아직 없음
    - IAM 정책
        
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:ListBucket"
                    ],
                    "Resource": [
                        "arn:aws:s3:::mele-bucket/tmp/"
                    ],
                    "Condition": {
                        "StringLike": { "s3:prefix": [ "tmp/*" ] }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "s3:DeleteObject"
                    ],
                    "Resource": [
                        "arn:aws:s3:::mele-bucket/tmp/*"
                    ]
                }
            ]
        }
        ```
        
- 새로 바뀐 EventBridge로 cron 돌리기
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/197f232b-2996-424f-be3d-b76d4f3dca33)
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/b5cfeb55-8efd-4e35-a7ed-a1da729021ce)
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/143ee250-e523-4acf-bdeb-5f34a301574c)
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/d7507434-96a0-412c-ae55-5073bdb26ad4)
    
    `다음` 버튼 클릭
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/31b6ad50-f487-4ebd-b1ab-02e029e4ba4d)
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/d9e62ebb-75a6-41cc-b820-b5605afbcf25)
    
    `다음` 버튼 클릭
    
    `일정 생성` 버튼 클릭
    

# 5장 요약

- AWS Lambda 함수를 사용하는 방법을 알아봄
- 라이브러리, 모듈, 바이너리를 함수와 함께 배포 패키지를 생성
    - Amazon S3와 같은 AWS 자원의 이벤트에 대한 반응으로 함수를 실행
    - 함수를 주기가 있는 일정에 따라 실행
        - Amazon EventBridge 사용

### TODO @Me1e

1. 5.3장 실습을 진행 못 함
    1. 최신 기술인 람다 계층(Layer) 기능도 사용해보며 구현 할 예정
    2. 최신 코드와 AWS 콘솔에서 구현하는 방법을 이 레포지토리에 업로드 할 예정
