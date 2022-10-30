# Blue-Green Deployments

- `recreate` 전략이라는 게 있다. 이건 New Pods 가 생성되기 전에 Old Pods 를 모두 제거하는 것이다. 

  - 주로 두 어플리케이션 사이의 communication-contract 가 변경될 때 사용한다. 이 경우에는 CD (Continuous Deployment) 전략이 유용하지 않기 떄문에.
  - 즉 모든 Replica 가 한번에 변경되어야함. 
  - 이런 전략은 `recreate strategy` 라고 부른다.

- blue-green 전략은 new pods 그룹을 모두 생성한 후 old pods 를 제거하지 않고 트래픽을 모두 old pod 에서 new pod 로 옮긴다. 
  - 이건 kubernetes 에 service 라는 네트워크 레벨을 추상화 해준 녀석 때문이다.
  - service 는 pod 에 들어갈 네트워크 트래픽 흐름을 조절해주는 역할을 한다. 그래서 여기에서 old pod 에서 new pod 로 바꿔서 가능해짐. 
  - 이게 가능해지는 이유는 label 을 통해서 구별하기 때문이다.    
  - 단점은 리소스를 잠시동안 두 배로 쓴다는 점이지만 한번에 새로운 버전의 앱을 출시하는게 가능하다는 점이다.  

#### service 

```yaml
kind: Service 
spec: 
  selector: 
    app: my-app
    version: v1 
```

- blue green 을 쓸러면 label 을 v2 로 만든걸 스케쥴 해놓고 서비스에 있는 version 을 v2 로 바꿔서 시랳ㅇ하면 된다.  

### Request and limit mechanism 

- 쿠버네티스에서 사용하고 있는 자원할당 매커니즘이다. 
- Request 는 최소로 필요한 자원량을 말하는 거고 limit 는 최대로 사용할 수 있는 자원 사용량을 말한다.
  - limit 가 request 보다 낮다면 에러를 낸다.
  - `kube-scheduler` 가 이 request 정보를 바탕으로 어떤 노드에다가 할당할지 정한다.
  - `kubelet` 은 limit 정보를 바탕으로 해당 컨테이너가 리소스를 limit 보다 적게 쓰도록 보장한다.    
- 이런 양은 per-pod 가 아니라 per-container 이다. 그래서 pod 가 여러개의 컨테이너를 포함한다면 각 컨테이너마다 리소스 사용량을 할당해줌. 
- cpu 를 정의할 떈 `millicores` 단위로 정의각 가능하다. 그러므로 만약 2개의 full core 가 필요하다면 `2000m` 이라고 하면됨.
- 만약 이 cpu 가 모든 node 가 가진 코어 개수보다 많다면 해당 pod 는 스케쥴 되지 않을 것이다.
- memory 는 byte 단위이다. 여기 예제에서는 MB 단위로 쓰였다. 

#### Container settings
```yaml
containers:
- name: container1
  image: busybox
  resources: 
    request:
      memory: "32Mi" 
      cpu: "250m"
    limits: 
      memory: "64mi"
      cpu: "500m" 
```