# Cluster Scaling

### Node Pool

- 먼저 GKE 에 있는 Node Pool 이 뭔지 간략하게 알고가자.
- Node Pool 은 GKE 에서 클러스터 내에 같은 설정을 공유하는 노드 그룹을 말한다.
  - 즉 지정된 노드 풀에 있는 모든 노드는 서로 동일하다. 
- 처음 GKE 를 생성할 때 노드 풀이 만들어지고 이후 클러스터에 노드풀을 추가하는게 가능하다. 
- 주로 노드 풀에서 설정할 수 있는 옵션으로 이런 것들이 있다. 
  - `local SSDs`
    - 높은 처리량, 낮은 지연시간을 가질 수 있다. 
    - 일시적인 저장소에 적합하다. local caching 이나 processing 작업에 특화. 
  - `minimum CPU platform`
    - CPU 집약적인 작업이 필요한 경우에 (i.e Gaming, Graphic 등) 최소한의 CPU 를 가진 플랫폼에서 작업이 돌아가도록 하는 것. 
    - 이를 통해서 더 퍼포먼스가 향상될 수 있다. 
  - `Spot VMs`
    - Auto Scaling 에 사용가능함. 
    - 일반적인 Virtual Machine 보다 저렴하다. 타입도 똑같은데 대신에 Computing Machine 이 해당 리소스를 요구한다면 그때는 사용가능하지 않음.
  - `node image`
    - 노드에 실행할 운영체제 이미지를 말함.
  - `machine types`
    - 머신의 타입. 
  - `virtual network interface`
    - GPU 기반의 어플리케이션의 경우 Network bandwidth 이 중요하다. 더 많은 요청을 보내기 떄문에. 
    - virtual network inteface (gVNIC) 를 통해서 bandwidth 를 올릴 수 있다.
- minimum size 와 maximum size 를 입력하는게 가능하다.
  - 특정 노드 풀을 스케일 다운으로 0 로 만드는 건 가능하지만 클러스터 전체에서 가능하진 않다. 최소한 하나의 노드는 있어야함.
  - 최대 15,000 개의 노드, 하나당 110개 pod 까지 테스트 했다.
- 각 노드풀에 auto scale 을 적용할 건지 여부도 설정할 수 있다.
    
## Cluster Scaling Introduction 

- GKE Cluster autoscaler 가 어떻게 동작하는지, 클러스터를 어떻게 설정하는지, 수동으로 클러스터 resize 하는 방법에 대해서 배운다.
- cluster 의 사이즈를 줄이는 경우 node 는 랜덤으로 줄어들고 이때 Pod 는 gracefully 하게 종료된다. 
  - graceful shutdown 은 TERM 시그널을 기반으로 종료되는 걸 말한다. 
  - 즉 KILL 신호가 보내져서 pod 가 삭제되기 전에 유예기간이 허용됨.
- Auto scaling 은 GKE Cluster 에 노드를 추가하는 것이다. 
  - 비용은 사용한 만큼만 나온다. 
  - workload 가 요구될 때 자동으로 됨. 
    - pod 를 만들어야 하는데 충분하 자원이 없을 때
    - 자원이 없어서 pod 를 만들지 못하는 경우에 pod 의 상태는 `unschedulable` 이 된다. cluster autoscaling 을 적용하면 gke autoscaler 가 이를 탐지해서 스케일링 한다.   
  - auto scaling 된 노드가 충분히 활용되지 않거나 다른 노드에 pod 를 배치할 수 있는 경우에 GKE 는 해당 노드를 삭제한다.
  - 이렇게 노드가 삭제될 때 pod 는 갑작스러운 종료를 경험할 수 있기 떄문에 해당 어플리케이션은 이에 대응할 수 있어야한다.  
  
## Downscaling 

- Cluster Autoscaler 는 scale-up event 가 pending 되는 걸 막는다.
  - 그래서 scale-down 중에 scale-up 이벤트가 발생한다면 scale-down 은 실행되지 않는다.
- Cluster Autoscaler 는 안전하게 노드가 제거될 수 있는지를 확인한다. 
  - 그래서 노드가 가지고 있는 파드가 다음의 조건에 해당될 경우 노드를 지우지 않는다.
    - controller 에 의해서 관리되지 않는 pod 인 경우 (deployment, replicaSet, statefulSet, job 등에 의해서 관리되지 않는 경우)
    - 로컬 스토로지를 가지고 있고, 다른 노드에서 실행되면 안되는 조건을 가진 pod 
    - pod 의 기본 세팅이 아닌 것 중 `safe-to-evict` annotation 이 false 로 등록된 pod. (이건 pod 레벨의 기본 설정으로 autoscaler 에게 제거되면 안된다고 말하는거임.)
    - PDB (Pod Disruption budget) 인 pod 인 경우. 
      - PDB 는 동시에 어플리케이션이 다운되는 수를 제한하는 용도로 쓰인다. 고가용성을 위해서 존재.
      - 예로 replica 가 5 고 pdb 값이 4라면 하나의 pod 의 disruption 만 허용한다.
      - 주로 부하를 견디기 위해서 최소한 있어야 하는 프론트엔드 앱이나 `quorum-based application` (의사결정 기반의 어플리케이션) 인 경우에 쓰면 좋다. 최소한의 의결권을 가진 어플리케이션이 있어야 하니까.
    - 지금껀 다 pod level 
- node level 에서 보자면 `nodes scale down disable annotation` 이 true 로 설정되어 있다면 해당 노드는 스케일 다운되지 않는다.
- scale-down 이 발생할려면 다음 조건을 모두 해당해야한다. (이 검사는 10초 주기로 한다.)
  - scale up 이 필요하지 않은 상황이어야 함.  
  - 할당한 pod 의 cpu + memory 합이 노드 리소스의 절반도 안되야함.
  - 노드에 있는 모든 pod 를 제거할 수 있어야함. (다른 노드에 reschedule 될 수 있어야함.)
  - 노드 레벨에서 `nodes scale down disable annotation` 이 true 로 설정되어 있으면 안됨. 
  - 이 모든 조건이 달성한 상태에서 10분동안 모니터링 한 후 종료한다. 
- auto scale 관련해서 best practice 를 보자. 
  - compute engine 에서 설정할 수 있는 auto scale 은 사용하지 말자. (GKE Auto scale 과 compute engine 의 auto scale 은 다름.)
  - cluster autoscaler 가 동작하고 있다면 수동으로 노드 풀 resize 하지 말자.
  - auto scale node 를 수동으로 변경하지 말자. (모든 노드는 같은 capacity, label, system part 를 가진다. 이걸 변경하지 말자는 뜻인듯.)
    - kubectl 을 통해서 특정 노드에 있는 label 을 바꾼다고 노드 풀에 있는 다른 노드에 있는 라벨도 바뀌진 않는다.
  - pod 에서 사용 가능한 리소스 양을 할당하자. (auto scale 에서 사용가능한 척도가 될 거니까. 이 값을 알지 못한다면 측정해야한다.)
  - pod disruption budget 을 사용하자. 고가용성을 위해서. 
    