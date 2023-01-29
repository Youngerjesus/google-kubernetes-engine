# More on PersistentVolumes

## Summary

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

- accessMode 에 따라서 Volume 이 어떻게 read 될 지, 어떻게 쓸 지를 결정해준다.
  - 각 PV 마다 accessMode 가 다를 수 있다.  
  - ReadWriteOnce
    - volume 이 Read-write 가 싱글 노드에게만 마운트 된다.
    - 이 노드가 k8s 노드를 말한다. pod 를 말하는게 아니다.
    - 그래서 동일한 pod 여러대가 같은 노드에 배치된다면 볼륨에 접근하는게 가능하다.  
  - ReadOnlyMany
    - Volume 이 마운트 된다. read-only 가 여러 노드에게. 
  - ReadWriteMany
    - Volume 이 read-write 가 여러 노드에게 가능해진다.
  - ReadWriteOncePod
    - 볼륨이 단일 pod 에서 읽기 쓰기 모드가 될 수 있다. 
  - GCP Persistent Disk 는 ReadWriteMany 를 지원하지 않는다. 
    - VolumeType 이 NFS 와 같다면 ReadWriteMany 를 지원한다. 
  - 대부분 어플맄케이션에서는 ReadWriteOnce 로 충분하다고함. 데이터가 static 이면 ReadOnlyMany 이고.

#### PersistentVolumeClaims manifest 

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: pd-volume-claim
spec:
  storageClassName: "standard"
  accessMode: 
  - ReadWriteOnce:
  resources: 
    requests:
      storage: 100G
  persistentVolumeReclaimPolicy: Retain
```

- pod 에서는 PV 를 쓰는게 아니라 PVC 를 써야한다. 

- 이제 Pod 를 작성하면 된다.

```yaml
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
    PersistentVolumeClaim: 
      claimName: pd-volume-claim 
```

- 이미 할당된 것보다 개발자가 더 많은 Storage 을 요구한다면 어떻게 될까?
  - 실패.
- 존재하지 않는 PersistentVolume 을 요청하게 되면 어떻게 될까? 
  - PVC 를 바탕으로 새로운 PV 을 만들어서 제공해줄려고 함.
  - 아. 이게 dynamic provisioning 이다. 클러스터에서 지원해주야함. 
- PVC 에 StorageClassName 을 입력하지 않으면 어떻게 될까? 
  - default StorageClassName 을 이용한다. GKE 에서는 standard.
- PVC 를 지우는 건 PV 를 지우는 것이다. 
  - 만약 PV 를 유지하고 싶다면 `persistentVolumeReclaimPolicy` 을 retain 으로 주자.
  - PV 가 필요하지 않을 때 PVC 를 지우면 된다. 
- PV 의 가용성을 늘리고 싶다면 어떻게 하면 될까? 
  - GCE Persistent Disk 를 regional persistent disk 로 배포하면 된다. 
  - Regional persistent disk 는 같은 region 기준으로 여러 zone 에 복제된다.
    - 이 설정은 수동으로, dynamic 하게도 가능하다.
  - StorageClass 에 `replication-type: regional-pd` 이 설정을 넣으면 된다.
- PV 를 pod 뿐 아니라 Deployments 와 StatefulSets 에 쓸 수도 있다.
  - deployment 에 쓸거면 accessMode 를 `ReadWriteMany` 나 `ReadOnlyMany` 써야겠지. 
    - `ReadWriteOnce` 를 쓰면 데드락에 걸릴 수 있다고한다. 
      - statefulset 은 괜찮다고 하는데 왜그럴까? 

#### Regional persistent disks

````yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd
  zones: us-central1-a, us-central1-b
````
