**이번 챕터에서는**

- 파드의 업데이트
- 롤링 업데이트
- 파드 롤백

### RC나 RS 사용 시 파드 업데이트 방법

- 기존 파드 모두 삭제 후 새 파드 시작
    - 중단 발생
- 새 파드 시작 후 전환, 기존 파드 제거
    - 기존 버전과 새 버전 공존으로 DB 스키마 등 공유하는 리소스 인터페이스가 변경되면 해저드 발생

### RC로 자동 롤링 업데이트

```
$ kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2
Command "rolling-update" is deprecated, use "rollout" instead
Created kubia-v2
Scaling up kubia-v2 from 0 to 3, scaling down kubia-v1 from 3 to 0 (keep 3 pods available, don't exceed 4 pods)
Scaling kubia-v2 up to 1
Scaling kubia-v1 down to 2
Scaling kubia-v2 up to 2
Scaling kubia-v1 down to 1
Scaling kubia-v2 up to 3
Scaling kubia-v1 down to 0
Update succeeded. Deleting kubia-v1
replicationcontroller/kubia-v2 rolling updated to "kubia-v2"
```

**안쓰는 이유**

- 파드와 RC의 레이블 셀렉터를 쿠버네티스가 수정
- 쿠버네티스 마스터 노드가 아닌 클라이언트의 요청으로 진행
    - 네트워크 에러 등으로 중단 될 경우 rollback이 되지 않는다
- 쿠버네티스가 최적의 방법을 결정하는 것이 아닌 세부 명령을 직접 수행

## 디플로이먼트

**왜 굳이 새로운 리소스를 추가?**

롤링 업데이트를 위해서는 추가 RS를 생성하고 두 컨트롤러가 잘 스케일링 되도록 관리가 필요

### **디플로이먼트 생성**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubia  # 이름에 버전정보를 명시할 필요가 없다
spec:
  replicas: 3
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
  selector:
    matchLabels:
      app: kubia
```

```
$ kctl create -f descriptor/ch09/deployment-v1.yaml --record
deployment.apps/kubia created

$ kctl get deployments,pods
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/kubia   3/3     3            3           10s

NAME                         READY   STATUS    RESTARTS   AGE
pod/kubia-66b4657d7b-7n287   1/1     Running   0          10s
pod/kubia-66b4657d7b-ptlq6   1/1     Running   0          10s
pod/kubia-66b4657d7b-rglmt   1/1     Running   0          10s
```

`--record` : kubectl 명령어를 리소스 애노테이션에 기록, 지정하지 않을 경우 저장된 내용이 있을 경우에만 업데이트

Record current kubectl command in the resource annotation. If set to false, do not record the command. If set to true, record the command. If not set, default to updating the existing annotation value only if one already exists.

파드의 이름은 다음과 같이 구성된다

`<디플로이먼트 이름>-<레플리카셋 해시값>-<파드 해시값>`

```
$ kctl get rs
NAME               DESIRED   CURRENT   READY   AGE
kubia-66b4657d7b   3         3         3       2m29s
```

### 디플로이먼트 업데이트

`kubectl rolling-update` 는 클라이언트에서 업데이트에 관련된 명령을 요청했기 때문에 모든 과정이 끝날 때 까지 터미널을 열어두고 기다려야 했다.

디플로이먼트는 리소스에 정의된 파드 템플릿만 수정하면 업데이트가 진행되고 선택할 수 있는 옵션은 아래와 같다.

- RollingUpdate (default)
    - 기존 파드를 하나씩 제거하고 새 파드를 추가한다
    - 이전 버전과 새 버전이 동시에 서비스된다
- Recreate
    - 이전 파드를 모두 삭제하고 새로 생성한다
    - 짧은 서비스 중단이 발생한다

**업데이트 할 이미지 변경 방법**

- 디플로이먼트 디스크립터 yaml 수정
- `kubectl patch deployment <DEPLOYMENT_NAME> -—patch '{"spec": {"template": {"spec": {"containers": [{"name": "CONTAINER_NAME","image": "NEW_IMAGE"}]}}}}'`
- `kubectl set image deployment <DEPLOYMENT_NAME> <CONTAINER_NAME>=<NEW_IMAGE>`

**(참고) 쿠버네티스 리소스 수정 방법**

- **kubectl edit** : 편집기가 실행되고 편집기를 종료하면 수정된 내용이 반영된다
- **kubectl patch** : 오브젝트 개별 속성을 수정한다
- **kubectl apply** : 전체 yaml, json 파일 속성값을 적용한다, 파일이 없으면 생성한다
- **kubectl replace** : apply와 동일하지만 기존 파일이 존재해야 한다
- **kubectl set image** : 디스크립터에 정의된 컨테이너 이미지를 변경한다

**(참고) 컨피그 맵 또는 시크릿은 수정하더라도 파드를 업데이트 하지 않는다. 이를 수정하려면 새 컨피그맵을 만들고 파드 템플릿이 새 컨피그 맵을 참조하는 방법밖에 없다.**

**롤링 업데이트 후 기존 RS이 남아 있다. 왜?**

### 디플로이먼트 롤백

kubectl 명령어를 통해 수동으로 마지막 rollout을 되돌릴 수 있다. rollout 프로세스가 진행중에도 사용이 가능하다.

`$ kubectl rollout undo deployment <DEPLOYMENT_NAME>`

**rollout history 확인**

디플로이먼트 생성 시 `--record` 옵션을 추가하지 않으면 CHANGE-CAUSE 열이 <none>으로 표시된다.

```
$ kctl rollout history deployment kubia
deployment.extensions/kubia
REVISION  CHANGE-CAUSE
1         kubectl create --filename=descriptor/ch09/deployment-v1.yaml --record=true
2         kubectl create --filename=descriptor/ch09/deployment-v1.yaml --record=true
```

**롤백**

```
# 이전 버전으로 롤백
$ kctl rollout history deployment kubia
deployment.extensions/kubia
REVISION  CHANGE-CAUSE
2         kubectl create --filename=descriptor/ch09/deployment-v1.yaml --record=true
3         kubectl create --filename=descriptor/ch09/deployment-v1.yaml --record=true

# 특정 버전으로 롤백
$ kctl rollout undo deployment kubia --to-revision=1
...
```

rollout 후 유지하고 있는 기존 RS를 통해 직전 버전 뿐만 아니라 제거되지 않은 모든 버전으로 롤백이 가능하다. 

다만 모든 이력을 유지하는 것은 너무 복잡하기 때문에 디플로이먼트 리소스의 editionHistoryLimit 속성에 따라 이전 RS를 삭제한다. (v1 기준 default: 10)

### Rollout 속도 제어

**롤링 업데이트의 strategy 속성**

```
spec:
	strategy:
		rollingUpdate:
			maxSurge: 1
			maxUnavailable: 0
		type: RollingUpdate
```

롤링 업데이트 시 추가되는 파드의 수는 maxSurge로 제거되는 파드의 수는 maxUnavailable로 설정할 수 있다. 지정하지 않으면 25%로 설정되며 백분율 혹은 절대값으로 지정할 수 있다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1f67888e-8334-4282-8b5d-00e56134c65e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1f67888e-8334-4282-8b5d-00e56134c65e/Untitled.png)

### Rollout 일시 정지

```
$ kctl sset image deployment kubia nodejs=luksa/kubia:v4
$ kctl rollout pause deployment kubia
...
업데이트 된 일부 파드가 정상동작하는지 확인
...
$ kctl rollout resume deployment kubia
```

당연히 이렇게 배포하는 것은 적절하지 않고 뒤에 다시 설명한다

### Rollout 정상 동작 확인

Rollout 중 파드에 문제가 있는 경우 중단해야 한다. minReadySeconds를 설정해 바로 배포하지 않고 지정된 시간 후에 파드가 준비되었는지 레디니스 프로브 결과로 확인한다.

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: kubia
spec:
  replicas: 3
  minReadySeconds: 10  # 
	...
    spec:
      containers:
      - image: luksa/kubia:v3
        name: nodejs
        readinessProbe:  # RedinessProbe 지정
          periodSeconds: 1
          httpGet:
            path: /
            port: 8080
```

### Rollout 데드라인

기본적으로 10분 동안 Rollout이 진행되지 않으면 실패로 간주한다. 이 경우 describe deployment로 ProgressDeadlineExceeded 상태를 확인할 수 있고 기간은 progressDeadlineSeconds 로 지정할 수 있다.

### Blue-Green 배포

![이미지](https://d33wubrfki0l68.cloudfront.net/5a87649bfab8bd84f95c288e8eb0f01c52274e12/7dd66/images/blog/2018-04-30-zero-downtime-deployment-kubernetes-jenkins/resources.png)
