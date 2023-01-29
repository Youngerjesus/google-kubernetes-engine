# Volume Type 2 

## 학습 목표 

- Persistent Storage 를 어떻게 쓸 수 있는지를 알자.
  - NFS, local, PersistentVolumeClaims 알면 될 듯. 

## Summary 

- secret 과 configMap 은 Pod 의 life 와 연결됨. 
- Secret 
  - sensitive 한 정보를 저장.
  - 데이터가 암호화된다. password 나 key 를 바탕으로.
  - 일시적인 파일 시스템으로 영구 저장은 아님.
- ConfigMap
  - 환경 변수, command-line 과 같은 데이터가 담김
  - 민감한 정보는 저장되지 않는다.
  - 이것도 일시적인 파일 시스템임.
- Volume 은 크게 두 종류가 있다. ephemeral 과 persistent

- Persistent Store 는 data-loss 를 해결해줄 수 있다.
  - 개발자는 `PersistentVolumeClaims` 통해서 사용하면 된다는 듯.
    - storage 의 양과 storage class 를 정해서 요청하고 운영팀에서는 이것들을 관리하고 
    - 이게 GKE 만 되는 기능인가? 클라우드 공급자만 제공해줄 수 있는 것 같은데.
      - 여러가지 volume type 이 있다. NFS 를 쓰면 될 것 같은데.  
      - Worker Node 에다가 기록하는건 에바인가? local 이라는 volume type 으로 존재함.
    - `PersistentVolumeClaims` 도 볼륨 유형 중 하나이다. 특징은 사용자가 특정 클라우드의 세부 환경을 모르고 볼륨을 클레임 할 수 있다는 것. 
      - GCE Persistent Disk or iSCSI 볼륨
    - `PersistentVolume` 은 스토로지 리소스임. `PersistentVolumeClaims` 는 사용자가 스토로지를 요청하는 것이고. 
  - pod-level 과 cluster-level 의 persistent store 가 있다.
  - Persistent Disk 는 Network based 의 storage 이다.
  - 일단 Google 에서는 100GB Compute Engine 의 Persistent Disk 를 만들려면 이렇게한다. 
    - `gcloud compute disks create --size=100GB --zone=us-central1-a demo-disk`
  - Pod 에서 사용하기 전에는 반드시 만들어야 한다고 한다. 만든 사람이 administration right 를 가지고 있고.
  - 그리고 구글에서 Persistent Disk 를 pod 에 할당할려면 이렇게 하면 된다. 

#### gcePersistentDisk example 

```yaml
...
spec: 
  containers:
  - name: demo-container
    image: gcr.io/hello-app:1.0
    volumeMounts:
    - mountPath: /demo-pod
      name: pd-volume
  volumes:
  - name: pd-volume
    gcePersistentDisk:
      pdName: demo-disk
      fsType: ext4
```

- K8s 1.17 에서 제거됨. 
- gcePersistentDisk 은 Google Compute Engine (GCE) 에서 영구 디스크를 Pod 에 마운트 할 때 쓴다.
- `pdName: demo-disk` 의 이름인 compute engine 이 만들어져 있어야한다.
- 이 방법은 옛날 방법이라함. ㅅㅂ 
- 요즘 방식은 node 에 붙이는 것 같은데. pod 가 다른 노드로 간다면 해당 노드에서 Detach 되고 다른 노드에 다시 attach 되는 방식.
- `kubectl describe pod` 명령으로 pod 에 volume 이 제대로 붙었는지 확인가능.
- 미리 데이터를 persistent disk 에 넣어두고 공유하는 것도 가능.
- GKE 는 Compute Engine Persistent Disk 를 사용한다.
- Persistent Disk 를 하드 코딩으로 특정 disk 를 입력하면 마이그레이션에서 어려움이 있을 것. 그래서 Persistent Volume abstraction 을 이용해서 Storage Pool 에서 할당 받아서 사용한다. 
