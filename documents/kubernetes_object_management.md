# Kubernetes Object Management

- 쿠버네티스에 있는 모든 object 들은 unique name and a unique identifier 으로 구별된다.
  - (이건 파드에 이름에도 해당되네. 유니크한 식별자가 붙어있으니)
- YAML or JSON 파일을 통해서 Object 의 상태를 정의해놓는 manifest 파일을 작성할 수 있다.
  - manifest 파일에 포함되야 하는 것으로 `apiVersion` 이 있는데 이건 object 를 만들 Kubernetes API 버전을 말한다.
    - 이걸 통해서 하위 호환성을 만족시킬 수 있다.
  - YAML 파일을 작성할 때 관련있는 object 들을 한 곳에 모아놓는게 좋다. 이게 베스트 프렉티스 중에 하나다.
    - 하나의 파일이 더 관리하기 쉽다는 취지.
  - object 의 이름은 유니크해야한다. 중복이 있으면 안된다. 이 오브젝트가 사라지면 이름을 재사용 하는건 괜찮다. 
  - label 은 오브젝트에 붙는 tag 로 object 가 만들어지는 중 또는 만들어지고 나서도 사용된다. 
    - label 을 통해서 오브젝트들을 식별하고, 조직화 하는게 가능하다. 
    - 레이블을 통해서 쿠버네티스 리소스를 선택하는 방법을 제공한다.
      - `kubectl get pods --selector=app=nginx` 이런식으로 라벨을 통해서 모든 파드를 조회할 수 있다. 
        - 이러면 쿠버네티스의 모든 리소스에게 물어보고 가져온다. 
- `pod` 는 self-healing, repair 와 같은 기능들은 없다. 그리고 pod 를 많이 띄우고 싶어서 YAML 을 많이 작성하는 것도 비효율적이다. 
  - 즉 pod 를 관리해줄 요소가 필요하다. 이것을 해주는 요소가 `Deployment`, `StatefulSets`, `DaemonSets`, `Jobs` 이렇게 있다.
  - `Deployment` 는 파드를 그룹으로 관리해주고 오래 생존하는 요소 (웹 서버 같은) 에 걸맞다.
    - `Deployment` 는 kube-scheduler 에 의해서 스케쥴된다. (이때 kube-apiserver 에 알린다.)
    - `Deployment` 가 파드들을 관리해줌. 
      - pod 의 rolling upgrade 를 도와준다. 
        - rolling upgrade 는 새로운 버전의 pod 를 출시하는 것. downtime 없이.  
        - 이것을 할 때 두 번째 Replica set 을 만들어서 pod 의 수를 늘리고 첫 번쨰 Replica set 의 pod 수를 줄이는 과정을 통해서 이룸. 
        - replicaSet 의 목적 자체는 파드의 상태를 유지하기 위해 있는 것. 
- pod 를 스케쥴링 할 때 중요한 건 리소스 할당이다. 제한된 리소스를 가진 노드에 올린다면 해당 파드가 메모리와 cpu 를 다쓰게 되고 죽을 것. 
  - 그러므로 memory 와 cpu 와 같은 리소스 사용량을 파드에 지정하는 것도 필요하다.
- namespace 를 통해서 pod, deployment, controller 들을 스코핑 하는게 가능하다. (single cluster 를 multiple cluster 로 만드는 효과.)
  - namespace 내에 리소스 할당량을 정해놓을 수 있다. 
  - 기본으로 만들어지는 네임스페이스는 `default` 와 `kube-system` 이 있다.  

## A note about Services 

- Service 는 pod 에 접근하기 위한 load-balancer 역할을 해준다.  
- Service 는 크게 세 가지 종료의 타입이 있다.
  - ClusterIP: 클러스터 내에서만 접근이 가능하다. (기본 타입임.)
  - NodePort: 클러스터 내의 각각의 노드에 대한 IP 를 들어낸다. (구체적인 port 번호와 함께)
  - LoadBalancer: Service 를 외부에 들어낼 떄 쓴다. 이때 로드 밸런서는 cloud provider 에 의해서 제공된다.
    - GKE 같은 경우에 로드 밸런서는 기본적으로 `regional Network Load Balancing configuration` 를 제공한다. 
      - Global HTTP(S) 요청을 받을려면 ingress 오브젝트를 이용해야한다.   

### Controller objects to know about 

- Kubernetes controller objects 들의 관계를 소개한다. 
  - ReplicaSets
  - Deployments
  - Replication Controllers
  - StatefulSets
  - DaemonSets
  - Jobs 
- Deployments 는 내부적으로 자신만의 Replicaset 을 관리한다.
  - 이를 통해서 create, update, roll back, scale pods, 등의 작업을 할 수 있다. 
  - 그 중 하나가 rolling update. 
  - Replication Controller 도 Deployment + Replicaset 의 역할을 수행하는데 더이상 쓰는걸 추천하진 않음. Deployment 를 써도 충분함. 
- StatefulSets 은 Deployment 와 비슷한 점이 많음. 다른 점은 상태를 관리할 수 있음. 
  - 이를 local state 라고 불리는데 StatefulSets 은 자신만의 Persistent Disk storage 나 stable network 를 가진다
  - deployment 는 이를 가지지 않음.  
  - (Stable Network ID 도 영구적으로 통신할 수 있는 주소가 필요해서 그런듯. data service 에 많이 이용한다. )
- DaemonSet 을 이용하면 클러스터 내에 특정 노드들이나 모든 노드에서 파드를 실행하는게 가능하다. 
  - 그래서 새로운 노드가 추가되더라도 조건에 맞으면 해당 파드를 실행할 수 있다. 
  - daemon 이라는 뜻처럼 non-interactive process 라는 의미를 가지고 있음.
  - 주로 logging (= fluentd) 와 같은 작업을 할 때 쓴다. 
- Job Controller 는 주어진 task 를 수행하기 위한 pod 를 만든다. 그리고 해당 task 가 수행이 완료되면 pod 를 종료하고. 
  - cronJob 이라는 것도 있는데 이건 스케쥴이 맞춰서 실행한다.  


#### example 
    
```yaml
apiVersion: app/v1 
kind: Deployment
metadata: 
    name: nginx
    labels: nginx
      app: nginx
      env: dev
      stack: frontend 
spec: 
    replicas: 3
    selector: 
      matchLabels:
      app: nginx
```