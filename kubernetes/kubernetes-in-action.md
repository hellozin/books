# Kubernetes In Action
Marko Luksa 지음

## 1부 쿠버네티스 개요

### 1장 쿠버네티스 소개

**쿠버네티스는 왜 쓸까?**

모놀로틱 서비스에서 마이크로 서비스로 옮겨가며 애플리케이션의 단위가 더 작고 단순해졌다. 하지만 그만큼 하나의 서비스가 더 많은 수의 애플리케이션으로 구성되면서 각 애플리케이션 간 관리 오버헤드가 발생하기 시작했다. 이러한 오버헤드를 줄이기 위해 K8S가 나오게 되지 않았을까?

마이크로 서비스의 주요 키워드는 독립 개발, 독립 배포 인 것 같다. 서비스를 나누는 단위도 배포 가능한 컴포넌트

**찾아보자**

- 모놀로틱 서비스가 마이크로 서비스로 바뀌면서 생기는 문제는 뭐가 있을까?
- 마이크로 서비스에서 REST API로 통신하는 것은 바로 떠오르는데 AMQP는 어떤 방식으로 사용되고 있을까?
- VM과 컨테이너의 차이는 뭘까?

### 2장 도커와 쿠버네티스 첫걸음

- 리눅스가 아닌 머신(맥이나 윈도우)에서 도커를 설치하면 리눅스 가상머신이 생성되고 가상머신 안에 도커 데몬이 구동된다.
- 도커의 빌드 프로세스는 도커 클라이언트가 아니라 도커 데몬에서 수행된다.
- 빌드 시 디렉터리 전체 파일이 데몬에 업로드 되기 때문에 데몬이 로컬로 실행중이지 않을 경우(맥이나 윈도우) 전체 파일 크기에 따라 업로드 시간이 길어질 수 있다.
- 빌드 디렉터리에 불필요한 파일을 추가하지 말자.
- `docker ps`보다 자세한 json 포멧의 결과를 보여주는 `docker inspect <container-name>`
- minikube : 로컬에서 간단하게 쿠버네티스를 조작해 볼 수 있는 단일 노드 클러스터 설치 도구
```
# 클러스터 시작
minikube start
```
>>> 나중에 다시 정리 !!!

### 3장 파드: 쿠버네티스에서 컨테이너 실행

#### 파드는 왜 필요할까?

컨테이너의 설계 목적 -> 단일 프로세스 (여러 프로세스 x)

#### 왜?

- 컨테이너는 관련이 없는 다른 프로세스는 관리(실패 시 재시작 등)하지 않는다.
- 모든 프로세스가 동일한 표준 출력으로 로그를 남겨 어떤 프로세스가 남긴 로그인지 파악이 어렵다.

#### 그렇다면

여러 프로세스 -> 여러 컨테이너, 즉 각 프로세스는 개별 컨테이너로 실행 

여러 컨테이너를 묶고 관리할 단위가 필요하다, **파드!**

파드를 통해 연관된 프로세스를 함께 실행하고 단일 컨테이너와 유사한 환경을 제공한다.

#### 어떻게?

같은 파드 내의 컨테이너는 부분 격리, 2장에서는 컨테이너가 서로 완벽하게 격리, 이제는 컨테이너 '그룹'이 서로 완벽하게 격리한다.

파드 내 컨테이너는 아래 내용을 공유하도록 설정한다.

- 리눅스 네임 스페이스
- 네트워크 네임스페이스
- UTS(UNIX Timesharing System) 네임스페이스

이를 통해 같은 호스트 이름, 네트워크 인터페이스를 공유한다.

다만 파일시스템의 경우 대부분 이미지에 종속되기 때문에 다른 컨테이너와 완전히 격리된다.  
이를 해결하기 위해 뒤에서 '볼륨'에 대해 알아본다!

---

#### 파드 내부의 네트워크

네트워크 네임스페이스가 같다  
-> 동일한 IP 주소와 포트 공간을 공유한다  
-> 파드 내부의 프로세스는 포트가 겹치지 않제 주의해야 한다.

또한 동일한 루프백 네트워크 인터페이스를 가지기 때문에 localhost를 통해 컨테이너 간 통신이 가능한다.

> 가상 루프백 인터페이스: 
> 같은 기기에서 동작하는 클라이언트와 서버는 가상 루프백 인터페이스를 통해 실제 패킷을 보내지 않고도 일반적인 네트워크 소프트웨어 스택으로 보내진다. 유닉스 계열 시스템에서는 보통 이것을 루프백 인터페이스 lo 혹은 lo0이라고 명한다.

쿠버네티스 클러스터 내의 모든 파드는 하나의 flat한 공유 네트워크를 사용해 NAT 없는 TCP 통신이 가능하다. 마치 LAN 처럼

#### YAML로 파드 생성하기

파드를 정의하는 Yaml 파일을 생성한다.

```yaml
# 쿠버네티스 API 버전 v1을 사용한다
apiVersion: v1

# Pod에 대한 명세이다
kind: Pod

# 관련 명세
metadata:
    name: simple-server-manual
spec:
    containers:
        - image: ianhellozin/simple-server
        name: simple-server
        ports:
            - containerPort: 8080 # 없어도 된다, 단지 표시용
            protocol: TCP
```

#### 로그

`kubectl logs POD_NAME`
or
`kubetcl logs POD_NAME -c CONTAINER_NAME`

#### 파드에 요청 보내기

로컬에서 파드에 직접 접근하는 방법 중 하나인 포트 포워딩

`kubectl port-forward POD_NAME 8888:8080`

#### 레이블

쿠버네티스 오브젝트에 레이블을 지정해 논리적인 그룹으로 관리할 수 있다.

레이블을 통해 파드를 특정 노드 그룹에 스케줄링 하는 식으로 활용할 수 있다.

#### 어노테이션

어노테이션은 레이블과 달리 셀렉터같은 것이 없다. 그저 정보를 보여주기 위해 사용한다.

#### 네임스페이스

레이블로 묶인 그룹은 겹칠 수 있다. 겹치지 않는 그룹으로 나누고 싶을 때 네임스페이스를 사용한다.

네임스페이스로 리소스를 격리해 관리할 수 있지만 실행중인 오브젝트의 격리도 보장하지는 않는다.

### 4장 레플리케이션과 그 밖의 컨트롤러: 관리되는 파드 배포

App crash는 쿠버네티스가 재실행 해 준다. 다른 오류나 버그가 있는 경우는 어떻게 알지?
(process가 실행되면 쿠버네티스는 컨테이너가 정상이라고 판단한다)

**liveness probe**

pod의 specification에 각 컨테이너 별 liveness probe를 지정할 수 있다.

**probe 실행 방법**

- HTTP GET probe : 지정한 IP:port/path 로 HTTP GET 요청, 2xx 3xx 응답 시 Success
- TCP socket probe : 컨테이너의 지정된 port에 TCP 연결 시도
- Exec probe : 임의의 명령 exit code가 0이면 Success

```docker
# HTTP GET probe
...
spec:
  containers:
    - image: image_name
      name: container_name
      livenessProbe:
        httpGet:
          path: /
          port: 8080
```

crash 된 컨테이너의 로그 확인하기

```docker
kubectl logs pod_name --previous
```

다시 시작한 이유 알아보기

```docker
kubectl describe po pod_name
...
Containers:
	Last State: ...
	Liveness: <probe_type> <url> delay=x timeout=x period=x #success=x #failure=x
Events: ...
```

delay, timeout, period 등은 probe 정의 시 지정할 수 있다.  
ex) initialDelaySecondes: 15

> ! probe가 너무 무거운 연산을 하지 않도록 주의하자.  
> probe의 리소스는 컨테이너에 포함되기 때문에 컨테이너 CPU 제한에 불필요한 영역을 차지하게 된다.

probe는 마스터 노드가 아닌 워커 노드의 kubelet에서 실행한다.

노드의 고장은 마스터가 처리한다. 따라서 고장 시 다른 노드에서 실행하는 부분은 레플리케이션 컨트롤러 또는 관련 매커니즘으로 파드를 관리해야 해결할 수 있다.

**레플리케이션 컨트롤러** : 파드가 항상 실행되도록 보장
- 노드가 사라진 경우
- 노드에서 파드가 제거된 경우  
    역할 : 레이블 셀렉터의 파드 수가 지정된 수와 일치하도록

**레플리케이션 컨트롤러의 3개 요소**
- 레이블 셀렉터
- 레플리카 수
- 파드 템플릿

레이블 셀렉터와 파드 템플릿은 변경되어도 기존의 파드에 영향을 주지 않는다.  
레플리케이션 컨트롤러를 정의할 때 (yaml) 레이블 셀렉터를 지정하지 않으면 자동으로 지정한다. 이를 통해 yaml을 더욱 간결하게 유지할 수 있다.

**레플리케이션컨트롤러 정보 가져오기**

`kubectl get rc`

파드의 label을 변경하면 레이블 셀렉터에 의해 확인되지 않기 때문에 레플리케이션컨트롤러가 새 파드를 실행한다.  
이를 통해 문제가 있는 파드를 레이블에서 제거해 테스트 해 볼 수 있다.

**레플리케이션컨트롤러 수정**

`kubectl edit rc replication_controller_name`

위 파일의 파드 템플릿을 수정하고 기존 파드를 제거하면서 파드 업데이트를 할수도 있지만 뒤에서 더 좋은 방법을 배운다

레플리케이션컨트롤러가 없어도 파드 접근이 가능한가?

#### 레플리카 셋

차세대 레플리케이션컨트롤러 > 이제 RC는 완전히 사용되지 않을것이다.  
일반적으로 직접 생성하지 않고 나중에 배울 디플로이먼트에 의해 자동으로 생성된다.

RC보다 풍부한 표현이 가능한 파드 셀렉터 사용  
- type=foo, type=val 동시에 관리 가능
- env=* 처럼 관리 가능

```docker
apiVersion: v1
kind: ReplicaSet
metadata:
	name: replica_set_name
spec:
	replicas: 3
	selector:
		matchLabels:
			previous: label_name
		template
			metadata:
				labels:
					previous: label_name
			spec:
				containers:
				- name: container_name
						image: image_name 
```

**파드 셀렉터**

```docker
selector:
	matchExpressions:
		- key: foo
			operator: In or NotIn or Exists or DoesNotExist
			values:
				- bar1
				- bar2...
```

각 노드, 혹은 특정 노드에 1개씩 배포하고 싶은 파드는 **데몬셋**으로 관리  
파드 셀렉터와 일치하는 파드 1개가 동작하는지 확인  
나머지는 레플리케이션컨트롤러, 레플리카셋과 동일하게 동작한다.

**Job** : 완료 가능한 태스크를 수행하는 파드  
컨테이너 재시작 x  
노드 장애 시 위와 동일하게 다른 노드에 스케줄링  
프로세스 자체 장애는 재시작 설정 가능  
`restartPolicy` 기본 값인 `always` 가 맞지 않기때문에 명시적으로 `OnFailure`나 `Never`로 명시 필요

> yaml 디스크립터에서 apiVersion: batch 이다. 리소스별로 달라지는 API 그룹을 잘 확인해야겠다.

job 파드가 완료되어도 삭제되지 않아 로그를 확인할 수 있다.  
`completions` : job을 몇번 수행할것인지  
수행할 때 마다 파드를 새로 생성한다

parallelism : job을 동시에 몇개 수행할것인지  
지정된 수만큼 파드를 생성한다  
일반 파드와 마찬가지로 실행 도중 스케일링이 가능하다

`kubectl scale job job_name --replicas 3`

- `activeDeadlineSeconds` : job에 타임아웃을 걸 수 있다
- `startingDeadlineSeconds` : cron 잡이 너무 늦게 시작하지 않도록 한다
- `spec.backoffLimit` : 잡 재시도 횟수
- `spec.schedule` : "cron format" 크론 잡으로 만들 수 있다
