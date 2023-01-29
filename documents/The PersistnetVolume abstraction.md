# The Persistent Volume abstraction 

## Summary  

- Persistent Volume 은 Network based 의 파일 시스템이다. 
- 크게 두 종류의 컴포넌트가 있다. 
  - PersistentVolume
    - Cluster Level 에서 관리된다. 
    - 스토로지 "리소스" 임. 
    - Manually or Dynamically provisioned 가능.
    - Persistent Volume 으로 GCE (Google Compute Engine) 의 Persistent Disk 를 사용하는 것도 가능. 
  - PersistentVolumeClaims
    - Pod 가 리소스를 사용하겠다는 request 임.
      - 요청에 담긴 정보들.  
      - Volume size 
      - access mode
      - StorageClass 
        - Storage 특징들의 집합 
    - PersitentVolueClaim 을 통해서 Persistent Volume 을 요청한다. 

- Pod-level Volume vs Cluster-level PersistentVolume
  - Pod-level 의 Volume 은 pod 안에 Volume 이 있는 것. 
  - Cluster-level 의 Volume 은 클러스터 안에 Persistent Volume 이 있고, Pod 가 PersistentVolumeClaims 를 통해서 PersistentVolume 을 요청하는 것.
  - PersistentVolume 은 어플리케이션 설정과 Storage 관리의 decoupling 을 위해서 나왔다. 
  - 

![](./images/pod-level%20volume%20and%20persistentVolume.png)

#### Pod manifest with a volume 

````yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: demo-pod 
spec: 
  containers:
  - name: demo-container
    images: gcr.io/hello-app:1.0
    volumeMounts: 
    - mountPath: /demo-pod
      name: pd-volume
  volumes:
  - name: pd-volume 
    gcePersistentDisk: 
      pdName: demo-disk 
      fsType: ext4
````

#### PersistentVolume manifest 

````yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pd-volume
spec: 
  storageClassName: "standard"
  capacity: 
    storage: 100G 
  accessMode:
  - ReadWriteOnce: 
  gcePersistentDisk: 
    pdName: demo-disk
    fsType: ext4
````

- StorageClass 를 통해서 PersistentVolume 을 구현한다. 
- Persistent Volume 에 있는 StorageClassName 과 PersistentVolumeClaims 의 StorageClassName 과 매칭이 되야한다. 
- GKE 의 StorageClass 의 기본형으로 "standard" 를 제공한다. GCE 의 Persistent Disk 를 사용하는 것.

#### GKE StorageClass: standard manifest 

````yaml
kind: StorageClass 
apiVersion: storage.k8s.io/v1
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: none 
````

#### StorageClass: ssd 

````yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pd-volume
spec: 
  storageClassName: "ssd"
  capacity:
    storage: 100G 
  accessModes: 
  - ReadWriteOnce:
  gcePersistentDisk:
    pdName: demo-disk
    fsType: ext4

---

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
````

- StorageClass 를 새로 만들 수도 있는 걸 보니 내가 만들고 싶은 Storage 에 이름만 부여하는 건가?
- GCE 에서는 기본적으로 standard 가 만들어져 있는 거고.
- k8s StorageClass 와 Google Cloud Storage Class 를 헷갈려하지마라고한다.
  - k8s Storage Class 는 PersistentVolume 을 지원하기 위한 선택. 
  - Google Cloud Storage Class 는 웹을 위한 Object Storage 이다.  
  - 아.. 이름이 비슷하니까. Google Cloud Storage Class 는 완전히 다른 거네.  

## Question 

### StorageClass 가 정확하게 뭔데? 
- 관리자가 제공하는 Storage 의 특징들을 이름으로 나타낸 것. ssd 를 제공한다면 ssd 라는 이름을 주는 것처럼. 
- 관리자가 정한 품질 수준, 백업 정책, 임의로 정한 정책과 매핑된다고 한다.
- PV (Persistent Volume) 을 구현해주는게 Storage Class 이다. 
- PV 를 동적으로 프로비저닝 할 때는 `provisioner`, `parameters` 와 `reclaimPolicy` 라는 필드가 사용된다. 
- StorageClassName 으로 어떤 PV 를 쓸 건지 요청한다. 이걸 지정해주지 않으면 기본 StorageClassName 인 것이 할당됨. 

- StorageClass 에서 PV 프로비저닝에 사용될 볼륨 플러그인이 있다. 이것들을 반드시 지정해줘야 한다고함.
  - 쿠버네티스에는 내장 NFS 프로비저너가 없다.

#### StorageClass: Local Provisioner 

````yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
````

- 로컬 볼륨은 동적 프로비저닝을 지원하지는 않는다.
- `WaitForFirstConsumer` 로 볼륨 바인딩 모드를 지연시키면 스케쥴러가 PVC (Persistent Volume Claims) 에 적절한 PV 를 선택할 때 모든 파드의 스케줄링 제약 조건을 고려할 수 있다고함. 
  - PVC 를 사용하는 파드가 생성될 때까지 PV 의 바인딩과 프로비저닝을 지연시키는 것. 
  - 이 모드 말고는 `Immediate` 모드가 있다. PVC 가 생성될 때 볼륨 바인딩과 동적 프로비저닝이 즉시 발생. 

#### StorageClass: NFS Provisioner

````yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-nfs
provisioner: example.com/external-nfs
parameters:
  server: nfs-server.example.com
  path: /share
  readOnly: "false"
````

- server: NFS 서버의 호스트 네임 or IP 주소
- path: NFS 서버가 export 할 경로 
- readOnly: 스토로지를 읽기 전용으로 마운트 할 건지 플래그. 기본값 false 

- StorageClass 의 Reclaim Policy 
  - 동적으로 생성된 PV 에서는 Reclaim Policy 를 가진다고 한다. 
  - Delete or Retain 이 될 수 있다. 기본 값은 Delete. 
  - 수동으로 생성한 PV 에서는 생성시 할당된 Reclaim Policy 가 있다고함. 

- StorageClass 의 볼륨 확장 허용 
  - PV 를 확장이 가능하도록 설정할 수 있다고함. 이 값을 true 로 설정해놓으면 PVC 오브젝트를 편집해서 볼륨 크기를 조정하는게 가능.

- StorageClass 의 마운트 옵션
  - StorageClass 에 의해 동적으로 생성된 PV 는 `mountOptions` 필드에 지정된 마운트 옵션을 가진다고 한다. 

- StorageClass 의 볼륨 바인딩 모드
  - `volumeBindingMode` 는 볼륨 바인딩과 동적 프로비저닝의 시작 시기를 제어한다. 
  - 기본 값은 `immediate`
    - 토폴로지 제약이 있거나 클러스터 모든 노드에서 전역으로 접근이 안되는 Storage 의 경우에는 파드와 연결이 안되서 파드가 스케쥴링이 안되는 문제가 생길 수 있다고함. 
  - 다른 값으로 `WaitForFirstConsumer` 가 있다. 

- StorageClass 의 허용된 토폴로지
  - 토폴로지가 뭐지?
    - 파드의 토폴로지 제약은 region, zone, node 같은게 있다는듯? 
  - `WaitForFirstConsumer` 를 쓴 상황에서는 대부분 특정 토폴로지로 프로비저닝을 제한할 필요는 없다고 한다. 
  - `allowedTopologies` 을 쓰면 특정 zone 으로 제한하는 방법이 가능하다고함. 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - us-central-1a
    - us-central-1b
```

- StorageClass 의 파라미터들.
  - 프로비저너가 사용할 파라미터들이다. 
  - 즉 프로비저너마다 다른 파라미터를 쓸 수 있다. 
    - AWS EBS 같은 경우는 `io1` 과 파라미터 `iopsPerGB` 를 쓸 수 있다. 이 파라미터의 설명에 대한 정보는 AWS 문서를 봐야지.

#### StorageClass: EBS 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```

- dynamic vs manually provisioning
  - dynamic 은 on-demand 에 따라서 만들어지는 것. 
  - manually 는 Storage Volume 을 미리 만들어 두는 것. 
  - StorageClass 는 만들어져있고 request 가 오면 provisioner 이 그떄그때 만드는 것.
