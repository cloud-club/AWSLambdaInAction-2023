# Chapter 9 - 인증 서비스 구현하기

**기존에 다뤘던 기술들을 함께 적용하는 조금 더 복잡성이 높은 실습을 진행해보자**

- 목표
  - 사용자 생성, 이메일을 통해 사용자 인증하기
- 사용 서비스
  - Lambda
  - Cognito
  - DynamoDB
  - SES
- 살펴볼 내용
    - 예시 인증 서비스를 위한 서버리스 아키텍처 구현하기
    - 람다 함수를 활용하여 백엔드 생성
    - 클라이언트 애플리케이션 구현을 위한 브라우저에서 구동되는 자바스크립트와 HTML 이용하기
    - AWS CLI를 활용한 초기화 자동화 및 배치
    - 복수의 람다 함수 간 코드 공유 및 통합 구성
    - 관리할 서버가 없는 상태에서 Amazon SES를 활용하여 이메일 보내기


## 통합 구성 관리하기
구성 정보, 설정 값과 코드는 분리하여 관리
코드와 구성 정보를 분리하는 것은 좋은 습관이다.

>따로 설정해서 쓰세요.

config.json
```yaml
{
  "AWS_ACCOUNT_ID": "123412341234",
  "REGION": "us-east-1",
  "BUCKET": "BUCKET",
  "MAX_AGE": "10",
  "DDB_TABLE": "SampleAuthUsers",
  "IDENTITY_POOL_NAME": "SampleAuth",
  "DEVELOPER_PROVIDER_NAME": "login.yourdomain.com",
  "EXTERNAL_NAME": "Sample Authentication",
  "EMAIL_SOURCE": "sender@yourdomain.com",
  "VERIFICATION_PAGE": "https://BUCKET.s3.amazonaws.com/verify.html",
  "RESET_PAGE": "https://BUCKET.s3.amazonaws.com/resetPassword.html"
}
```

## 초기화 자동화 및 배치
init.sh를 사용하면 모든 리소스 생성, 초기화가 진행 가능
- Lambda Function 6개 생성
  - Lambda Function 마다 지정해줄 IAM 정책 6개
- 프로필 정보 저장을 위한 DynamoDb
- 인증 서비스를 위한 Cognito identity pool
  - 인증을 테스트할 IAM 정책 2개 (인증 가능 IAM / 인증 실패 IAM)
  - Lambda Function 과 역할을 위한 Trust Policy
- 배포에 사용된 HTML, JavaScript 파일을 저장하기 위한 S3 bucket

init.sh
```shell
#!/bin/bash

# Check if the AWS CLI is in the PATH
found=$(which aws)
if [ -z "$found" ]; then
  echo "Please install the AWS CLI under your PATH: http://aws.amazon.com/cli/"
  exit 1
fi

# Check if jq is in the PATH
found=$(which jq)
if [ -z "$found" ]; then
  echo "Please install jq under your PATH: http://stedolan.github.io/jq/"
  exit 1
fi

# Read other configuration from config.json
AWS_ACCOUNT_ID=$(jq -r '.AWS_ACCOUNT_ID' config.json)
CLI_PROFILE=$(jq -er '.CLI_PROFILE' config.json)
# Get jq return code set by the -e option
CLI_PROFILE_RC=$?
REGION=$(jq -r '.REGION' config.json)
BUCKET=$(jq -r '.BUCKET' config.json)
MAX_AGE=$(jq -r '.MAX_AGE' config.json)
DDB_TABLE=$(jq -r '.DDB_TABLE' config.json)
IDENTITY_POOL_NAME=$(jq -r '.IDENTITY_POOL_NAME' config.json)
DEVELOPER_PROVIDER_NAME=$(jq -r '.DEVELOPER_PROVIDER_NAME' config.json)

#if a CLI Profile name is provided... use it.
if [[ $CLI_PROFILE_RC == 0 ]]; then
  echo "Setting session CLI profile to $CLI_PROFILE"
  export AWS_DEFAULT_PROFILE=$CLI_PROFILE
fi

# Create S3 Bucket
aws s3 mb s3://$BUCKET

# Create DynamoDB Tables
echo "Creating DynamoDB Table $DDB_TABLE begin..."
aws dynamodb create-table --table-name $DDB_TABLE \
    --attribute-definitions AttributeName=email,AttributeType=S \
    --key-schema AttributeName=email,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
		--region $REGION
echo "Creating DynamoDB Table $DDB_TABLE end (creation still in progress)"

# Create Cognito Identity Pool
IDENTITY_POOL_ID=$(aws cognito-identity list-identity-pools --max-results 1 \
	  --query 'IdentityPools[?IdentityPoolName == `'$IDENTITY_POOL_NAME'`].IdentityPoolId' \
	  --output text --region $REGION)
if [ -z "$IDENTITY_POOL_ID" ]; then
	echo "Creating Cognito Identity Pool $IDENTITY_POOL_NAME begin..."
	IDENTITY_POOL_ID=$(aws cognito-identity create-identity-pool --identity-pool-name $IDENTITY_POOL_NAME \
	    --allow-unauthenticated-identities --developer-provider-name $DEVELOPER_PROVIDER_NAME \
	    --query 'IdentityPoolId' --output text --region $REGION)
	echo "Identity Pool Id: $IDENTITY_POOL_ID"
	echo "Creating Cognito Identity Pool $IDENTITY_POOL_NAME end"
else
  echo "Using previous identity pool with name $IDENTITY_POOL_NAME and id $IDENTITY_POOL_ID"
fi

# Updating Cognito Identity Pool Id in the configuration file
mv config.json config.json.orig
jq '.IDENTITY_POOL_ID="'"$IDENTITY_POOL_ID"'"' config.json.orig > config.json
rm config.json.orig

cd iam
if [ -d "edit" ]; then
  rm edit/*
else
  mkdir edit
fi

# Create IAM Roles for Cognito
for f in $(ls -1 Policy_Trust_*); do
  echo "Editing trust from $f begin..."
  sed -e "s/<AWS_ACCOUNT_ID>/$AWS_ACCOUNT_ID/g" \
      -e "s/<DYNAMODB_TABLE>/$DDB_TABLE/g" \
      -e "s/<DYNAMODB_EMAIL_INDEX>/$DDB_EMAIL_INDEX/g" \
      -e "s/<REGION>/$REGION/g" \
      -e "s/<IDENTITY_POOL_ID>/$IDENTITY_POOL_ID/g" \
      -e "s/<REGION>/$REGION/g" \
      $f > edit/$f
  echo "Editing trust from $f end"
done
for f in $(ls -1 Policy_Cognito_*); do
  role="${f%.*}"
  echo "Creating role $role from $f begin..."
  sed -e "s/<AWS_ACCOUNT_ID>/$AWS_ACCOUNT_ID/g" \
      -e "s/<DYNAMODB_TABLE>/$DDB_TABLE/g" \
      -e "s/<DYNAMODB_EMAIL_INDEX>/$DDB_EMAIL_INDEX/g" \
      -e "s/<REGION>/$REGION/g" \
      -e "s/<IDENTITY_POOL_ID>/$IDENTITY_POOL_ID/g" \
      -e "s/<REGION>/$REGION/g" \
	      $f > edit/$f
  if [[ $f == *_Unauth_* ]]; then
    trust="Policy_Trust_Cognito_Unauth_Role.json"
    unauthRole="$role"
  else
    trust="Policy_Trust_Cognito_Auth_Role.json"
    authRole="$role"
  fi
  aws iam create-role --role-name $role --assume-role-policy-document file://edit/$trust
  aws iam update-assume-role-policy --role-name $role --policy-document file://edit/$trust
  aws iam put-role-policy --role-name $role --policy-name $role --policy-document file://edit/$f
  echo "Creating role $role end"
done
echo "Setting identity pool roles begin..."
roles='{"unauthenticated":"arn:aws:iam::'"$AWS_ACCOUNT_ID"':role/'"$unauthRole"'","authenticated":"arn:aws:iam::'"$AWS_ACCOUNT_ID"':role/'"$authRole"'"}'
echo "Roles: $roles"
aws cognito-identity set-identity-pool-roles \
  --identity-pool-id $IDENTITY_POOL_ID \
  --roles $roles \
  --region $REGION
echo "Setting identity pool roles end"

# Create IAM Roles for Lambda Function
for f in $(ls -1 sampleAuth*); do
  role="${f%.*}"
  echo "Creating role $role from $f begin..."
  sed -e "s/<AWS_ACCOUNT_ID>/$AWS_ACCOUNT_ID/g" \
      -e "s/<DYNAMODB_TABLE>/$DDB_TABLE/g" \
      -e "s/<DYNAMODB_EMAIL_INDEX>/$DDB_EMAIL_INDEX/g" \
      -e "s/<IDENTITY_POOL_ID>/$IDENTITY_POOL_ID/g" \
      -e "s/<REGION>/$REGION/g" \
      $f > edit/$f
	trust="Policy_Trust_Lambda.json"
  aws iam create-role --role-name $role --assume-role-policy-document file://edit/$trust
  aws iam update-assume-role-policy --role-name $role --policy-document file://edit/$trust
  aws iam put-role-policy --role-name $role --policy-name $role --policy-document file://edit/$f
  echo "Creating role $role end"
done

cd ..

cd fn

# Create Lambda Functions
for f in $(ls -1); do
  echo "Creating function $f begin..."
  cp ../config.json $f/
  cp -R ../lib $f/
  cd $f
  zip -r $f.zip index.js config.json lib/
  aws lambda create-function --function-name ${f} \
      --runtime nodejs16 \
      --role arn:aws:iam::"$AWS_ACCOUNT_ID":role/${f} \
      --handler index.handler \
      --zip-file fileb://${f}.zip \
	  	--region $REGION
	sleep 1 # To avoid errors
  cd ..
  echo "Creating function $f end"
done

cd ..

./deploy.sh
```

AWS API 설명

>aws s3 mb s3://$BUCKET
- S3의 신규 bucket 생성
  - bucket명 (BUCKET) config 파일에서 설정
  - https://docs.aws.amazon.com/cli/latest/reference/s3/mb.html

> aws dynamodb create-table --table-name $DDB_TABLE \ 
--attribute-definitions AttributeName=email,AttributeType=S \
--key-schema AttributeName=email,KeyType=HASH \
--provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
--region $REGION
- Dynamodb 테이블 생성
  - Attribute
    - email (String) -> Primary Key로 사용
  - 읽기 저리 용량 / 쓰기 처리 용량: 5
    - Region은 config 파일에서 설정
  - https://docs.aws.amazon.com/cli/latest/reference/dynamodb/create-table.html

> aws cognito-identity create-identity-pool --identity-pool-name $IDENTITY_POOL_NAME \
--allow-unauthenticated-identities --developer-provider-name $DEVELOPER_PROVIDER_NAME \
--query 'IdentityPoolId' --output text --region $REGION
- Cognito Identity pool 생성
  - 비인증 사용자도 Identity pool Access 가능
  - https://docs.aws.amazon.com/cli/latest/reference/cognito-identity/create-identity-pool.html

> aws iam create-role --role-name $role --assume-role-policy-document file://edit/$trust
- IAM Role 생성
  - https://docs.aws.amazon.com/cli/latest/reference/iam/create-role.html

> aws iam update-assume-role-policy --role-name $role --policy-document file://edit/$trust
- IAM Trust Policy 업데이트
  - https://docs.aws.amazon.com/cli/latest/reference/iam/update-assume-role-policy.html

> aws iam put-role-policy --role-name $role --policy-name $role --policy-document file://edit/$f
- IAM Role에 정책 설정 업데이트
  - https://docs.aws.amazon.com/cli/latest/reference/iam/put-role-policy.html

> aws lambda create-function --function-name ${f} \
--runtime nodejs16 \
--role arn:aws:iam::"$AWS_ACCOUNT_ID":role/${f} \
--handler index.handler \
--zip-file fileb://${f}.zip \
--region $REGION
- Lambda Function 생성
  - fn 폴더 밑에 있는 Lambda Function들 모두 생성
  - config에서 설정한 Account Id의 Role 사용
  - https://docs.aws.amazon.com/cli/latest/reference/lambda/create-function.html

### 배포

deploy.sh
```shell
#!/bin/bash

# Check if the AWS CLI is in the PATH
found=$(which aws)
if [ -z "$found" ]; then
  echo "Please install the AWS CLI under your PATH: http://aws.amazon.com/cli/"
  exit 1
fi

# Check if jq is in the PATH
found=$(which jq)
if [ -z "$found" ]; then
  echo "Please install jq under your PATH: http://stedolan.github.io/jq/"
  exit 1
fi

# Read other configuration from config.json
REGION=$(jq -r '.REGION' config.json)
CLI_PROFILE=$(jq -er '.CLI_PROFILE' config.json)
# Get jq return code set by the -e option
CLI_PROFILE_RC=$?
BUCKET=$(jq -r '.BUCKET' config.json)
MAX_AGE=$(jq -r '.MAX_AGE' config.json)
IDENTITY_POOL_ID=$(jq -r '.IDENTITY_POOL_ID' config.json)
DEVELOPER_PROVIDER_NAME=$(jq -r '.DEVELOPER_PROVIDER_NAME' config.json)

#if a CLI Profile name is provided... use it.
if [[ $CLI_PROFILE_RC == 0 ]]; then
  echo "Setting session CLI profile to $CLI_PROFILE"
  export AWS_DEFAULT_PROFILE=$CLI_PROFILE
fi

echo "Updating Lambda functions..."

cd fn

# Updating Lambda Functions
for f in $(ls -1); do
  echo "Updating function $f begin..."
  cp ../config.json $f/
  cp -R ../lib $f/
  cd $f
  zip -r $f.zip index.js config.json lib/
  aws lambda update-function-code --function-name ${f} \
      --zip-file fileb://${f}.zip \
	  	--region $REGION
  cd ..
  echo "Updating function $f end"
done

cd ..

echo "Updating www content begin..."

cd www
if [ -d "edit" ]; then
  rm -r edit/*
else
  mkdir edit
fi
mkdir edit/js

for f in $(ls -1 *.* js/*.*); do
  echo "Updating $f begin..."
  sed -e "s/<REGION>/$REGION/g" \
      -e "s/<IDENTITY_POOL_ID>/$IDENTITY_POOL_ID/g" \
      -e "s/<DEVELOPER_PROVIDER_NAME>/$DEVELOPER_PROVIDER_NAME/g" \
      $f > edit/$f
  echo "Updating $f end"
done
echo "Updating www content end"
echo "Sync www content with S3 bucket $BUCKET begin..."
cd edit
aws s3 sync . s3://$BUCKET --cache-control max-age="$MAX_AGE" --acl public-read
cd ../..
echo "Sync www content with S3 bucket $BUCKET end"
```
AWS API 설명
>  aws lambda update-function-code --function-name ${f} \
--zip-file fileb://${f}.zip \
--region $REGION
- Lambda Function update
  - fn 폴더에 있는 함수들로 업데이트
  - https://docs.aws.amazon.com/cli/latest/reference/lambda/update-function-code.html

> aws s3 sync . s3://$BUCKET --cache-control max-age="$MAX_AGE"
- S3를 로컬과 동기화
  - 현재 경로(.) 아래 파일들을 S3 버킷과 비교하여 변경된 것 있으면 업로드
  - https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html

코드 출처: https://github.com/danilop/AWS_Lambda_in_Action

# 업데이트 해야 하는 부분

최신 node.js 코드로 업데이트는 못 함

1. init.sh
    1. 더 이상 지원하지 않는 `nodejs4.3` 대신, → `nodejs16` 으로 바꿔줘야 함
2. deploy.sh
    1. S3 정책이 바뀌어서 그런지(정확하지 않음) `-acl public-read` 를 붙이면 안됨
    
    <img width="600px" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/6636b06f-5784-4cb4-a075-824c12afb92a" alt="image" />
    
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
  
        <img width="600px" src="https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/46004dbb-4b67-4f8f-8cd9-4bb7e841303c" alt="image" />
        
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
