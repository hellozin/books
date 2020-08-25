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

---

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

---

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

---

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

### 5장 서비스 : 클라이언트가 파드를 검색하고 통신을 가능하게 함

**파드에 직접 접근하는게 부적합한 이유**

- 파드는 일시적, 노드의 공간 확보, replicas 변경, 노드 장애 등으로 언제든지 사라질 수 있다.
- 파드의 IP주소는 스케줄링 된 후 파드가 시작되기 직전에 할당되어 미리 알 수 없다.
- 수평 스케일링 된 동일한 기능을 하는 파드는 구분없이 하나의 엔드포인트를 통해 접근이 가능해야 한다.

**그럼 어떻게 접근할 수 있나?**

서비스

**외부 클라이언트 - 고정된 IP 주소 - 서비스 - 파드**

**서비스는 지원하는 파드를 어떻게 알아낼까?**

레이블 셀렉터

**서비스를 쉽게 생성하는 방법**

kubectl expose

**yaml descriptor를 사용해서 서비스 생성하기**

```yaml
# kubia-service.yaml
apiVersion: v1
kind: Service
metadata:
	name: kubia
spec:
	ports:
	- port: 80
		targetPort: 8080
	selector:
		app: kubia
```

`kubectl create -f kubia-service.yaml`

**서비스 확인하기**

```bash
$ kubectl get svc
NAME   CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
kubia  10.111.248.14  <none>       80/TCP   6m
```

Cluster-IP는 클러스터 내부에서만 접근할 수 있다.

**외부에서 실행중인 컨테이너에 명령어 실행**

`kubectl exec <pod-name> -- curl -s <service-cluster-ip>`

`--` : kubectl 명령어의 마지막을 의미 (like `';'`), `--` 뒤의 내용은 파드 내부에서 실행된다.

`-s, --silent` : Silent  or  quiet mode. Don't show progress meter or error messages.  Makes Curl mute. It will still output the data you ask for, potentially even to the terminal stdout unless you redirect it.

**💡DSR 설정에 변화는 없을까?**

**세션 어피니티**

하나의 서비스에 연결된 파드가 여러개일 경우 디스크립터에서 세션 어피니티를 설정할 수 있다.

```bash
spec:
	sessionAffinity: ClientIP # 동일한 client 요청은 동일한 pod로 전달된다.
```

사용 가능한 옵션은 None or ClientIP

k8s 서비스는 HTTP 수준이 아닌 TCP, UDP 패킷을 처리하기 때문에(페이로드는 신경쓰지 않는다.) 쿠키 기반 세션 어피니티 옵션은 없다.

**💡k8s 서비스의 HTTP와 TCP 이해하기**

**멀티 포트 서비스**

서비스는 단일 포트만 노출하지만 멀티 포트 서비스를 통해 여러 포트를 지원할 수도 있다.

```yaml
# kubia-multi-port-service.yaml
...
spec:
	ports:
	- name: http
		port: 80
		targetPort: 8080
	- name: https
		port: 443
		targetPort: 8443
	selector:
		app: kubia
```

멀티 포트 서비스를 만드려면 포트 이름(name)을 지정해야 한다.

레이블 셀렉터는 서비스 전체에 적용되기 때문에 포트 별 셀렉터를 구성할 수는 없다. 

**포트 번호 대신 지정된 이름으로 접근하기**

서비스 스펙을 좀 더 명확하게 하기 위해 파드에서 포트 이름 지정

```yaml
kind: Pod
spec:
	containers:
		- name: kubia
		ports:
		- name: http
			containerPort: 8080
		- name: https
			containerPort: 8443
```

서비스에서 지정된 포트 이름 사용

```yaml
kind: Service
spec:
	ports:
	- name: http
		port: 80
		targetPort: http
	- name: https
		port: 443
		targetPort: https
```

**클라이언트 파드에서 서비스의 IP를 알아내는 방법**

- **환경변수**

    k8s 시작 시점에 각 서비스를 가리키는 환경변수 세트를 초기화한다. 이를 통해 파드가 생성되기 전 존재하는 서비스 정보를 환경변수를 통해 알 수 있다.

    ```bash
    $ kubectl exec <pod-name> env
    ..
    KUBERNETES_SERVICE_HOST=10.123.12.13
    KUBERNETES_SERVICE_PORT=443
    ..
    KUBIA_SERVICE_HOST=12.123.11.123
    KUBIA_SERVICE_PORT=80
    ```

    이렇게 IP, Port를 찾는 대신 보통 DNS를 사용하지 않을까? 아래에서 살펴본다.

- **DNS**

    kube-system 네임스페이스에 kube-dns라는 서비스가 존재한다.

    클러스터 내 모든 파드는 자동으로 해당 DNS를 사용한다. (dnsPolicy로 사용 여부 설정 가능)

    그래서 서비스의 이름을 알고 있는 클라이언트 파드는 환경변수 대신 FQDN으로 접근이 가능하다.

- **FQDN**

    <서비스이름>.<네임스페이스>.<클러스터.도메인.접미사>

    kubia.default.svc.cluster.local

    이 경우 80(HTTP), 5432(Postgres) 등 표준 포트를 사용하지 않는 경우 환경변수에서 포트를 따로 가져와야 한다.

    통신하려는 파드가 동일한 네임스페이스에 있는 경우 네임스페이스와 접미사는 생략할 수 있다.

    파드 컨테이너 내부에 DNS resolver가 처리 (`/etc/resolv.conf`)

**서비스 IP 대신 DNS 사용**

- `container $ curl http://kubia.default.svc.cluster.local`
- `container $ curl http://kubia.default`
- `container $ curl http://kubia`

같은 네임스페이스에 있을 경우 위 세가지 모두 가능

파드의 컨테이너 내 shell 실행

`kubectl exec -it <container-name> bash`

💡 서비스의 클러스터 IP가 가상 IP이기 때문에 서비스 포트와 결합된 경우만 의미가 있어 ping은 동작하지 않는다. 그래서 서비스 헬스체크를 핑으로 하면 엉뚱한 결과를 얻을 수도 있다. (11장에서 알아본다.)

**클러스터 외부 서비스와 연결**

서비스는 파드에 직접 연결되지 않는다.

서비스의 파드 셀렉터를 통해 엔드포인트 목록을 생성한다.

```
$ kubectl describe svc kubia
Name: kubia
...
Selector app=kubia
...
Endpoints: <파드 IP>.<포트>, <파드 IP>.<포트>...
```

엔드포인트 또한 리소스이기 때문에 kubectl get 으로 조회할 수 있다.

```
$ kubectl get endpoints kubia
NAME   ENDPOINT       AGE
kubia  10.100.1.0:80  1h
```

파드 셀렉터는 서비스로 들어오는 요청에 직접 관여하지 않고 미리 엔드포인트 리소스를 만들며 서비스 프록시가 사용할 수 있도록 한다.

**엔드포인트와 서비스 분리**

엔드포인트는 서비스와 분리해 별도로 관리할 수 있다.

서비스에 파드 셀렉터를 지정하지 않고 엔드포인트 리소스를 서비스와 같은 이름으로 만들면 기존과 같이 사용할 수 있다.

❓파드셀렉터도 지정하고 엔드포인트 리소스도 생성하면?

```
...
kind: Endpoints
metadata:
	name: external-service
subsets:
	- address:
		- ip: 1.2.3.4
		- ip: 1.1.1.1
	ports:
	- port: 80
```

잘 모르겠다

내부 파드 > 서비스 > 외부 파드(엔드포인트)?

책: 외부 서비스를 k8s 내 파드로 마이그레이션 할 경우 서비스에 셀렉터를 추가해 엔드포인트를 자동으로 관리 가능. > 서비스 내 구현이 달라져도 엔드포인트는 동일하게 유지 가능

**FQDN으로 더 쉽게 외부 서비스 참조하기**

서비스 리소스의 spec.type을 ExternalName으로 설정한다.

[api.somecorp.com](http://api.somecorp.com) 라는 공개 API가 있다고 가정

```
...
kind: Service
metadata:
	name: external-service
...
spec:
	type: ExternalName
	externalName: api.somecorp.com # 실제 서비스의 FQDN
...
```

책에서는 [someapi.somecorp.com](http://someapi.somecorp.com) 으로 돼있던데 오타?

api... 대신 external-service(.default.svc.cluster.local)로 외부 서비스에 연결할 수 있다.

잘 모르겠다

나중에 externalName 을 변경하거나 유형을 ClusterIP로 변경하고 서비스 스펙을 만들어 스펙을 수정하면 나중에 다른 서비스를 가리킬 수 있다.

ExternalService는 CNAME DNS 레코드를 생성하고 서비스 프록시를 완전히 무시한 채 외부 서비스로 연결되기 때문에 ClusterIP를 얻을 수 없다.

**여기까지가 클러스터 내부 파드 > 서비스 > 내부 혹은 외부에 대한 내용**

**외부 > 서비스 > 내부 파드**

몇가지 방법

- NodePort: 노드 자체에서 포트 오픈, 해당 포트의 트래픽을 서비스로 전달
    - 서비스 > 내부 IP & 포트 뿐만 아니라 모든 노드의 포트로도 접근 가능
- LoadBalancer: k8s 내 프로비저닝 된 LB로 서비스 접근 가능
    - 클라이언트 > 로드밸런서 IP > 모든 노드의 노드포트
- 인그레스: 단일 IP로 여러 서비스 노출 가능
    - 뒤에서 설명한다.

**NodePort**

모든 노드에 특정 포트(모든 노드가 동일)를 할당한다.

ClusterIP와 유사하지만 서비스 내부 클러스터 IP 뿐만 아니라 노드의 IP, 포트도 사용할 수 있다는 차이점이 있다

```
kind: Service
spec:
	type: NodePort
	ports:
	- port: 80
		targetPort: 8080
		nodePort: 30123 # 생략하면 임의로 지정
```

```
$ kubectl get svc service-name
NAME      CLUSTER-IP  EXTERNAL-IP  PORT(S)     
svc-name  1.2.3.4     <nodes>      80:30123/TCP
```

서비스에는 다음과 같이 접근할 수 있다.

- 1.2.3.4:80
- <노드1 IP>.30123
- <노드2 IP>.30123 ...

**외부 클라이언트 > 노드:노드포트 > 서비스 > 파드**

노드 1 IP > .. > 노드 2 파드 가 될수도 있다.

+) 노드포트에 접근하려면 노드에 포트 오픈 및 방화벽 설정을 해주어야 한다.

모든 노드 IP 가져오기

`$ kubectl get nodes -p jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'`

- items 속성의 모든 항목을 조회한다.
- 각 항목의 status 속성을 조회한다.
- addresses 에서 ExternalIP 타입만 필터링한다.
- address를 가져온다

어떤 노드에 요청을 보내도 상관없지만 HA를 위해 노드들 앞에 LB를 붙이는게 좋다.

**로드밸런서 서비스 생성**

```
kind: Service
spec:
	type: LoadBalancer
	ports:
	- port: 80
		targetPort: 8080
		# NodePort 지정 x, (해도 된다)
```

로드밸런서는 노드포트 서비스의 확장

스펙에서 특정 노드포트를 지정하지 않으면 쿠버네티스가 포트를 선택한다

**외부 클라이언트 > 로드밸런서 > 노드포트 > 서비스 > 파드**

위에서 이야기한 방화벽 설정도 필요 없다.

**노드포트로 외부 연결 시 주의사항**

노드 포트로 서비스에 접속할 경우 다른 노드의 파드가 선택되면 추가 홉이 발생한다.

spec.externalTrafficPolicy: Local을 통해 현재 노드의 파드에만 트래픽을 전달한다.

대신 파드에 부하가 균등하게 분배되지 않는다 ex) 2노드 3파드 파드 A : B : C = 50 : 25 : 25

클러스터 IP 보존 불가

클러스터 내 클라이언트 > 서비스 연결 시 노드포트는 SNAT(소스 네트워크 주소 변환)가 발생하 클라이언트 IP를 알아야하는 app의 경우 문제가 될 수 있다(예를들면 req log)

위 local external traffic policy는 이문제를 해결한다

**인그레스**

- 노드밸런서 : 서비스 = 1 : 1
- 인그레스 : 서비스 = 1 : N
    - 요청한 host와 path에 따라 서비스를 분기할 수 있다
        - [a.host.com/A](http://a.host.com/A) > service AA
        - [b.host.com/A](http://b.host.com/A) > service BA
        - [b.host.com/B](http://b.host.com/B) > service BB

인그레스는 HTTP 계층에서 동작하기때문에 서비스에서 지원하지 못하던 쿠키 기반 세션 어피니티도 지원 가능하다.

인그레스를 사용하기 위해서는 클러스터에 인그레스 컨트롤러를 실행해야 한다.

```
kind: Ingress
spec:
	rules:
	- host: kubia.example.com # 인그레스가 이 도메인 이름을 서비스에 매핑한다
		http:
		paths:
			- path: /
				backend:
					serviceName: kubia-nodeport
					servicePort: 80
```

[kubia.example.com](http://kubia.example.com) 요청은 kubia-nodeport 80포트로 전달된다.

인그레스의 IP로 도메인을 연결하자

`$ kubectl get ingresses`

**인그레스가 필요한 경우**
