# 6장 볼륨: 컨테이너에 디스크 스토리지 연결 

**파드의 파일 시스템**

파드 내부의 컨테이너들은 CPU, RAM, 네트워크 인터페이스 등을 공유하지만 **파일 시스템**은 컨테이너 이미지를 기반으로 제공되기 때문에 컨테이너 별로 독립적이다.

컨테이너가 삭제되어도 데이터를 유지하고 싶다면? **스토리지 볼륨**을 사용할 수 있다.

**스토리지 볼륨**

파드와 라이프사이클을 함께하며 스토리지 볼륨을 통해 라이브니스 프로브에 의해 재시작된 파드가 종료된 컨테이너가 기록한 데이터를 확인할 수도 있고 여러 컨테이너가 공유하는 데이터를 만들 수 도 있다.

파드와 라이프사이클을 함께하지만 볼륨 유형에 따라 파드와 볼륨이 사라진 후에도 볼륨의 파일이 유지되어 새로운 볼륨으로 마운트 될 수 있다.

볼륨은 접근하려는 컨테이너에서 각각 마운트 해야한다. 또한 볼륨을 초기화하면서 외부 소스를 추가하거나 기존의 디렉터리를 마운트 할 수도 있다. 이 과정은 컨테이너가 시작되기 전에 수행된다.

**볼륨 유형**

- **emptyDir**: 일시적인 데이터 저장에 사용되는 간단한 빈 디렉터리
- **hostPath**: 워커 노드의 파일시스템을 파드의 디렉터리로 마운트
- **gitRepo**: Git 저장소의 컨텐츠로 초기화 (Deprecated)
- **nfs**: NFS로 공유되는 내용을 파드에 마운트
- **configMap, secret, downwardAPI**: 쿠버네티스 리소스나 클러스터 정보를 파드에 노출하는데 사용되는 틀별한 유형의 볼륨
- **persistentVolumeClaim:** 프로비저닝된 퍼시스턴트 스토리지 사용
- 그외 [볼륨 유형들](https://kubernetes.io/ko/docs/concepts/storage/volumes/#%EB%B3%BC%EB%A5%A8-%EC%9C%A0%ED%98%95%EB%93%A4)..

하나의 파드에서 동시에 여러 볼륨 유형을 사용하고 컨테이너별로 필요한 볼륨만 마운트 할 수 있다.

**자세히 알아보자 emptyDir**

하나의 파드 내에서 컨테이너 간 파일 공유하거나 컨테이너의 메모리로는 부족한 큰 데이터의 처리 등 임시 데이터를 저장하는데 유용하다.

후자의 경우 컨테이너 자체 파일시스템을 이용할 수도 있지만 상황에 따라 쓰기가 불가능한 경우도 있다. (뒤에서 설명한다)

`html` 볼륨을 공유하는 두개의 컨테이너 생성

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune
spec:
  containers:
  - image: luksa/fortune  # 10초 마다 임의의 문단으로 html파일을 생성
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    emptyDir: {}
```

`spec.volumes`에 **html**이라는 이름으로 정의된 **emptyDir** 볼륨을 각 컨테이너에서 `/var/htdocs`, `/usr/share/nginx/html` 로 마운트한다.

파드를 생성하고 port-forward나 서비스로 접근해보면 시간에 따라 변경되는 html을 확인할 수 있다.

위 **emptyDir**은 워커 노드의 **실제 디스크**에 생성되어 디스크 유형에 따라 성능이 결정된다. emptyDir을 메모리기반의 tmpfs로 생성하려면 아래 설정을 추가한다.

```yaml
volumes:
  -name: html
    emptyDir:
      medium: Memory
      sizeLimit: 1Gi
```

tmpfs는 매우 빠르지만, 디스크와 다르게 노드 재부팅 시 tmpfs가 지워지고, 작성하는 모든 파일이 컨테이너 메모리 제한에 포함된다.

**gitRepo (Deprecated) 생략**

emptyDir을 생성하고 Git 저장소를 해당 디렉토리에 복제한다.

**자세히 알아보자 hostPath**

파드는 기본적으로 호스트 노드를 인식하지 못하기 때문에 노드의 파일시스템에는 접근하면 안된다. 하지만 특정 시스템 레벨의 파드(보통 데몬셋)는 노드 디바이스 접근을 위해 노드의 파일시스템을 사용해야 한다.

**hostPath**를 사용하면 노드의 파일시스템 특정 파일이나 디렉토리에 접근할 수 있다.

당연히 하나의 노드 내 파드들은 hostPath를 통해 파일이나 디렉토리를 공유할 수 있다.

또한 이전 파드와 동일한 노드에 스케줄링된다면 파드가 삭제되어도 이전 파드에서 작성된 내용을 이어서 사용할 수 있다.

동일한 노드에 파드를 스케줄링 하려면 [nodeSelector](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/assign-pod-node/#%EB%85%B8%EB%93%9C-%EC%85%80%EB%A0%89%ED%84%B0-nodeselector)를 사용하는게 좋다.

다만 hostPath를 데이터베이스 디렉토리로 사용할 생각이라면 다시 고려해볼 필요가 있다. 파드는 꼭 필요한 경우에만 노드를 지정하자.

kube-system 네임스페이스의 파드를 살펴보면 hostPath를 데이터 저장이 아닌 노드 데이터 접근용으로 사용하는 것을 확인 할 수 있다.

**퍼시스턴트 스토리지 사용**

파드에서 실행중인 애플리케이션이 디스크에 데이터를 유지하고 파드가 다른 노드로 재스케줄링된 경우에도 동일한 데이터를 사용해야 한다면 지금까지 언급한 볼륨 유형은 사용할 수 없고 NAS 유형을 사용해야 한다.

**GCE(Google Compute Engine) 퍼시스턴트 디스크**

쿠버네티스 클러스터가 있는 zone에 GCE 퍼시스턴트 디스크를 생성한다.

```
$ gcloud compute disks create [options...] mongodb
...
NAME    ZONE       SIZE_GB ...
mongodb europe...  ...
```

GCE PD(Persistent Disk)를 사용하는 파드를 생성한다.

```yaml
kind: Pod
spec:
  volumes:
  - name: mongodb-data
    gcePersistentDisk:
      pdName: mongodb  # 생성된 PD의 이름과 일치해야 한다.
      fsType: ex4
containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
```

생성된 파드로 데이터를 저장하고 파드 삭제 후 재생성된 파드에서 이전 데이터를 확인할 수 있다.

💡 kakao 내 Persistent Disk도 있을까?

**NFS 볼륨**

외부 NFS 서버의 export path를 지정하면 마운트 할 수 있다.

```yaml
volumes:
- name: mongodb-data
  nfs:
    server: 1.2.3.4     # NFS Server IP
    path: /export/path  # NFS 서버의 export path
```

**그외 여러 스토리지**

ISCSI 디스크 리소스를 위한 iscsi, GlusterFS 마운트를 위한  glusterfs, RADOS 블록 디바이스를 위한 rdb 등 필요한 내용은 [공식 문서](https://kubernetes.io/ko/docs/concepts/storage/volumes/#%EB%B3%BC%EB%A5%A8-%EC%9C%A0%ED%98%95%EB%93%A4)를 확인하자.

그런데 위와 같은 인프라 접근 방법은 복잡하기도 하고 클러스터와 파드 간에 불필요한 의존성도 높이게 된다.

이를 해결하기 위해 다음 내용을 살펴보자.

**기반 스토리지 기술과 파드 분리**

**퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임**

**인프라 관리자 시점**

위에서 퍼시스턴트 디스크를 생성했다면 퍼시스턴트 볼륨을 추가한다.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity: 
    storage: 1Gi         # PV의 사이즈
  accessModes:           # 읽기/쓰기 & 읽기전용 으로 사용될 수 있다.
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain  # 클레임이 해제된 후 볼륨을 삭제하지 않고 유지한다.
  gcePersistentDisk:     # 위에서 생성한 PD 정보를 참조한다.
    pdName: mongodb
    fsType: ext4
```

옵션에 대한 자세한 내용은 [공식문서](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/)를 확인하자.

- accessModes
    - ReadWriteOnce -- 하나의 노드에서 볼륨을 읽기-쓰기로 마운트할 수 있다
    - ReadOnlyMany -- 여러 노드에서 볼륨을 읽기 전용으로 마운트할 수 있다
    - ReadWriteMany -- 여러 노드에서 볼륨을 읽기-쓰기로 마운트할 수 있다
- persistentVolumeReclaimPolicy
    - Retain(보존) -- 수동 반환
    - Recycle(재활용) -- 기본 스크럽 (`rm -rf /thevolume/*`)
    - Delete(삭제) -- AWS EBS, GCE PD, Azure Disk 또는 OpenStack Cinder 볼륨과 같은 관련 스토리지 자산이 삭제됨

```yaml
$ kubectl create -f /path/to/persistent-volume
$ kubectl get pv
...
```

퍼시스턴트 볼륨은 노드와 같은 클러스터 수준의 리소스로 네임스페이스에 속하지 않는다.

**개발자 시점**

퍼시스턴트 볼륨 클레임을 생성한다.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc  # 나중에 파드의 볼륨을 요청할 때 사용된다.
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: ""  # '동적 프로비저닝'에서 배운다. 빈 문자열이 아닐 경우 새로운 볼륨을 프로비저닝 하지 않고 미리 프로비저닝 된 볼륨에 바인딩 한다.
```

클레임을 생성하면 쿠버네티스가 적절한 퍼시스턴트 볼륨을 찾아 바인딩 해준다.

퍼시스턴트 볼륨 클레임이 잘 바인딩 됐는지 확인해보자

```
$ kubectl get pvc
NAME         STATUS  VOLUME      ...
mongodb-pvc  Bound   mongodb-pv  ...
```

바인딩이 되면 퍼시스턴트 볼륨의 STATUS도 Available에서 Bound로 바뀌는 것을 확인할 수 있다.

생성된 퍼시스턴트 볼륨 클레임을 사용해 파드를 생성한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb 
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:   # 위에서 만든 퍼시스턴트 볼륨 클레임을 참조한다.
      claimName: mongodb-pvc
```

**사용한 파드와 퍼시스턴트 볼륨 클레임을 재생성하면 어떻게 될까?**

```yaml
$ kubectl delete pod mongodb
$ kubectl delete pvc mongodb-pvc
$ kubectl create -f /path/to/pvc
STATUS가 Bound가 아닌 Pending으로 표시된다.
$ kubectl get pv
STATUS가 Available이 아닌 Released로 표시된다.
```

이미 볼륨을 사용했기 때문에 데이터를 가지고 있어 관리자가 볼륨을 완전히 비우지 않으면 새로운 클레임에 바인딩 할 수 없다. (persistentVolumeClaimPolicy가 Retain인 경우)

**Retain**

볼륨을 재사용하려면 볼륨 리소스를 삭제하고 다시 생성하는 방법밖에 없다.

**Recycle**

이 정책은 더 이상 사용하지 않는다. 대신 '동적 프로비저닝'을 권장한다.

볼륨의 콘텐츠를 삭제하고 다시 클레임 될 수 있도록 한다.

**Delete**

기반 스토리지를 삭제한다.

**퍼시스턴트 볼륨의 동적 프로비저닝**

앞의 내용들을 통해 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임으로 개발자가 내부 저장소 기술에 대한 고려 없이 쉽게 저장소를 얻을 수 있다는 것을 알게 되었다.

하지만 여전히 클러스터 관리자는 실제 스토리지를 미리 프로비저닝 해둬야 한다.

다행히 퍼시스턴트 볼륨의 '**동적 프로비저닝**'을 통해 이를 자동으로 수행할 수 있다.

1. 퍼시스턴트 볼륨 생성 대신 퍼시스턴트 볼륨 프로비저너 배포
2. 스토리지 클래스 오브젝트 정의
3. 퍼시스턴트 볼륨 클레임에서 스토리지 클래스를 참조
4. 프로비저너가 퍼시스턴트 스토리지를 프로비저닝

쿠버네티스는 인기있는 프로비저너를 포함하고 있어 별도 배포(1) 이 필요 없을 수 있다.

다만 온프레미스에서는 커스텀 프로비저너가 배포되어야 한다.

관리자가 여러 퍼시스턴트 볼륨을 미리 프로비저닝하는 대신 하나 혹은 그 이상의 스토리지 클래스를 정의해 클레임 요청마다 새로은 볼륨을 생성할 수 있고 스토리지 용량이 충분한 이상 볼륨이 부족할 일은 없어진다.

**스토리지 클래스 정의**

스토리지 클래스 리소스는 클레임이 스토리지 클래스에 요청할 때 어떤 프로비저너가 볼륨을 프로비저닝 할 지 정하는데 사용된다.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd  # 프로비저닝에 사용되는 볼륨 플러그인
parameters:                        # provisioner로 전달되는 파라미터
  type: pd-ssd
```

**퍼시스턴트 볼륨 클레임에서 스토리지 클래스 요청**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc 
spec:
  storageClassName: fast  # 위에서 정의한 스토리지 클래스를 참조한다.
  resources:
    requests:
      storage: 100Mi
  accessModes:
    - ReadWriteOnce
```

위 클레임을 생성하면 fast라는 이름의 스토리지 클래스에서 참조하는 프로비저너가 볼륨을 생성한다.

프로비저너는 수동으로 생성된 볼륨과 클레임을 매핑하는데도 사용된다.

만약 존재하지 않는 스토리지 클래스를 참조하면 프로비저닝은 실패한다.

kubectl describe <pvc> 를 확인해보면 ProvisioningFailed 이벤트가 표시된다.

클레임에서 스토리지를 이름으로 참조하기 때문에 필요한 경우 같은 클레임으로 다른 스토리지를 정의해 사용할 수 있다.

위에서 **fast** 라는 스토리지를 생성했는데 `$ kubectl get storageclass` 로 확인해보면 **standard(default)** 라는 스토리지가 이미 있는 것을 확인할 수 있다.

`$ kubectl get sc standard -o yaml` 로 자세한 내용을 살펴보면

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "true"
    name: standard
    ...
```

**standard** 가 default class로 지정되어 있어 클레임에서 명시적으로 스토리지를 지정하지 않은 경우 이 스토리지가 사용된다.

**마무리**

파드에 퍼시스턴트 스토리지를 연결하는 최적의 방법은 storageClassName을 지정한 PVC와 이 PVC를 이름으로 참조하는 파드만 생성하는 것이다.

그러면 동적 퍼시스턴트 볼륨 프로비저너가 처리해 준다.
