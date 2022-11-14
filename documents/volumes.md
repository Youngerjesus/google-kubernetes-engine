# Volumes

## Introductions 

- K8s 가 제공해주는 다양한 Storage Abstraction 을 살펴보는 곳.
  - 그게 바로 Volumes 과 Persistent Volumes
    - 이 둘이 어떻데 다른지, 어떻게 사용하는지 차이를 보면 된다. (pod 의 정보를 저정하고, pod 끼리 정보를 어떻게 공유하는지의 차이를 보면 된다는듯.)
    - Volumes 의 주 사용 목적이 pod 의 state 를 저장하거나, pod 내에 잇는 컨테이너 사이의 정보를 공유할 때 사용한다.
      - 컨테이너 내의 디스크 파일은 컨테이너가 크래쉬 되면 다 사라지는 문제를 가지고 있음. Volumes 는 그런 문제를 겪지 않는다.

## Volumes

- Pod 안에 있는 모든 컨테이너가 접근할 수 있는 디렉토리를 말한다.
  - 어떠한 Volumes 은 ephemeral 하다. (일시적임.) 그래서 연결된 pod 가 있는 경우에만 존재한다. (pod 에 붙어서만 살 수 있다. 이런 뜻인듯.) 
  - 예시로는 config map 과 empty directory 가 있다. 
- 또 어떤 Volumes 은 Persistent 하다.
- 모든 Volumes 은 Container 에 붙는게 아니라 pod 에 붙는다.
  - pod 가 node 에 매핑되지 않는다면 volumes 도 pod 에 매핑되지 않는다고 하는 거 같은데, 일부 volumes 은 persistent 특성이 있지 않나? 
  - 매핑되지 않을 뿐, 클러스터가 변하거나 pod 가 recreate, delete, terminate 되더라도 Persistent Volumes 은 독립적으로 존재한다. 
    - Persistent Volumes 은 Persistent Disk 와 연결된다. (cluster 가 변해도 상관없다고 하네.)
    - 생성은 cluster administrator 나, persistent volumes claim 에 의해서 이뤄지는듯.
  - GKE 에서 Persistent Volumes 는 NFS 라는 File Solution 을 이용한다고 한다. 이건 Google Cloud Solution 인듯.
    - NFS 는 Network File System 이란 뜻으로 remote computer 에 저장을 하는 것을 말한다. 그리고 마치 로컬처럼 사용하는 것. 

### Ephemeral Volumes 

- Ephemeral Volume 은 pod 와 라이프사이클이 같다.
  - pod 가 노드에 할당될 때 생기고 pod 가 종료될 때 사라진다.
- emptyDir Volume 이라는게 있다. 
  - 이 Volume 은 pod 내 container 에서 read 나 write 가 가능하다.
    - 해당 volume 은 어플리케이션 별로 같은 부분을 이용해서 공유하는 것도 가능하고 어플리케이션마다 서로 다른 부분을 쓰는 것도 가능.
  - 초기에는 비어있어서 emptyDir 이라는 이름임.
  - 이러 종류의 volume 은 지속되는 데이터를 저장하는 용도로는 적합하지 않다.
  - 이 Volume 은 node 의 local disk 나 memory-backed file system 으로 이뤄진다. (memory 기반의 파일 시스템이라고 생각하면 될려나)
  - 주 사용 목적은 급조한 결과물이 필요할 때 사용한다. 
    - disk-based merge
    - checkpointing a long computation 
    - recovery from crashes
    - for holding files that a content manager container fetches while a web server container serves the data.
- ConfigMap 이라는 것도 잇다. 
  - 이건 pod 안에 있는 application 에 설정 정보들을 넣을 때 사용한다.
  - configMap 에 있는 여러 설정 정보들은 Volumes 에서 참조된다. 마치 파일, 디렉토리 트리 구성처럼.
- Secret 이라는 것도 있다. 
  - ConfigMap 과 같지만 좀 더 sensitive information 을 저장할 때 사용한다. 
  - password, token, ssh 등.
  - Secret 은 in-memory file system 에서만 유지된다.. 그래서 비휘발성 스토로지에 쓰여지진 않는다.
  - Secret 에는 data or stringData 에 값은 Base64 encoding 기법을 써야하기도 한다. (올바른 데이터를 전달하기 위해서 그런건가. 일단 k8s 에서는 알아서 디코딩해줌.)
- download API 라는 것도 있다.
  - container 가 pod environment 에 대해서 알고 싶을 때 사용하는 것. 
  - 예로 들면 컨테이너화된 어플리케이션에서 유니크한 식별자를 받고 싶을 때 pod 의 이름을 쓰면 된다.
- ConfigMap, Secret, Download API 모두 emptyDir 기반의 volume 이다.

#### emptyDir Example 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec: 
  containers:
  - name: web
    image: nginx
    volumeMounts: 
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

