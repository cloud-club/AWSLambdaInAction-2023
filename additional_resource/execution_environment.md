# 실행 환경(Execution Environment)

![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/eb2ef24a-6d82-4aeb-9160-e9d61af3226a)

![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/ec6bdd83-7cef-4af5-8b1e-7392720a5c28)
## 소개

- Lambda가 코드를 실행하기 위해 생성한 격리된 런타임 환경
- 각 Lambda 함수 간에 공유되지 않으며 다른 AWS 계정과도 공유되지 않음
    - 1 함수 요청 = 1 실행 환경
        - 30개의 동시 요청은 30개의 실행 환경이 필요함
    - 단, 연속적인 함수 실행의 경우 실행 환경이 공유될 수 있음
- Lambda 실행 정보(메모리, 실행시간 등)을 기반으로 실행 환경을 구성
    - Lambda 함수가 업데이트될 때마다 새로 생성
- Firecracker 라는 MicroVM에서 실행
    - Firecracker: AWS에서 퍼블릭 클라우드 인프라에 맞는 격리 환경을 제공하기 위한 가상화 기술

## 구성 요소

- Memory: 128 mb ~ 10,240 mb
- CPU: 메모리에 연동되어(비례) 변동 → up to 6 vCPU
- Ephemeral Storage(임시 스토리지): 512 mb ~ 10,240 mb
- Timeout: 최대 15분
- Handler
- Provisioned Concurrency
- 환경변수

## 생명 주기

![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/b949e9c4-9655-4018-93a0-b5321b32d924)

### Init phase

![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/314ce98c-6485-4e3d-930f-4048f1573319)

- 실행 환경을 구성하는 단계
    - Extension init: Lambda에 추가 기능(커스텀 모니터링, 보안 등)을 실행
    - Runtime init: 언어 별로 다른 코드 실행을 위해 필요한 요소들 준비
    - Function init Code: 함수에서 필요한 정적 코드들을 준비
- 과금 되지 않음
    - Billed duration에 포함되지 않음
- 10초 제한
    - 10초 이상 걸린다면 이 단계를 건너뛰고 handler 호출
    - 이후 handler 호출 단계에서 함수 실행 시간 동안 init을 수행
        - 이때는 과금이 됨
        - 10초 안에 무료 init을 꼭 하자!
    - SnapStart가 활성화되어 있으면, Lambda Version이 배포된 시점에 실행
        - SnapStart: 속도가 중요한 Lambda의 시작 속도를 빠르게 구성해 주는 모드

### Invoke phase

- 실제 Lambda 함수를 수행하는 단계
- Lambda 함수의 총 수행 시간에 따라 Invoke 단계의 수행시간이 결정됨
- Invoke 단계에서 실패(crash)할 경우, 실행 환경은 리셋됨
    - 즉, Shutdown 단계로 넘어감
    - 그리고 다음 사이클에서 Init을 시도(suppressed init)
        
        ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/e2c34020-3bc3-4eb7-b802-b24c50947fec)
        
        ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/af70c6ce-815d-4211-915b-c2351632c21e)
        
    - suppressed init의 경우 log에 나타나지 않음
        - billing duration과 실제 실행 시간이 다름
            
            ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/1502214c-16ee-4f76-b9bf-1b7b37062d66)
            

### Shutdown phase

- 실행 환경을 정리하는 단계
- 2초 이후에는 sigkill 강제 종료
- 이후 어느 정도 시간동안 다음 함수 요청을 위해 실행 환경을 유지
    
    ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/b5bc14da-ad28-46f7-9eef-f74ef1a0c903)
    
    - 즉
        - 핸들러 바깥에서 정의되거나 초기화된 오브젝트의 경우 다음 함수에서도 남아있을 수 있음
        - DB Connection의 경우 다음 함수에서 재사용 가능할 “수도” 있음
        - 실행 환경에 속해 있는 /tmp 디렉토리 안에 내용이 남아있을 “수도” 있음
        - 백그라운드 프로세스/callback 중 이번 함수에서 종료되지 않은 경우 다음 함수에서 다시 실행되거나 수행될 수 있음
    - 항상 실행 환경이 유지된다고 확신 금지
        - 확인 후 initialize 로직 구현 필요
    - demo
        - Node.js / Python 기반 Lambda 함수 중, 일부로 완료되지 않은 promise를 수행
        - 이후 다음 invoke에서 이전의 Promise가 수행되는 것을 확인
        - Lambda 함수 코드
            ```jsx
            const sleep = (ms) => {
              return new Promise((r) => {
                setTimeout(r, ms);
              })
            }
            
            const excuteLater = async (ms, id) => {
              const date = new Date()
              console.log(`실행 아이디: ${id}, ${formatDateToHHMMSSMS(date)}에 시작`);
              await sleep(6000);
              console.log(`실행 아이디: ${id}, ${formatDateToHHMMSSMS(date)}에 끝`);
            }
            
            export const handler = async (event) => {
              const { id } = event;
              
              excuteLater(6000, id);
              
              await sleep(5000);
              
              return true;
            };
            
            const formatDateToHHMMSSMS = (date) => {
                let minutes = date.getMinutes().toString().padStart(2, '0');
                let seconds = date.getSeconds().toString().padStart(2, '0');
                let milliseconds = date.getMilliseconds().toString().padStart(3, '0');
                
                return `${minutes}:${seconds}.${milliseconds}`;
            }
            ```
        - 위 함수를 두 번 실행시켰을 때의 AWS CloudWatch 로그
          ![image](https://github.com/cloud-club/AWSLambdaInAction-2023/assets/76844285/1a7f64f9-635c-47fb-b12d-9268f51c0e4c)

# 여기서 다 베껴옴

https://youtu.be/IVrWb9n2p4E?si=oSEs99AE8h9DFvIY  
https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/lambda-runtime-environment.html
