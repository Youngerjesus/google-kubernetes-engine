# Updating Deployments

- Deployment 를 업데이트 해야할 경우가 생긴다. 

  - 새로운 버전의 pod 를 rollout 할 때 (pod 의 image 를 변경 시킬 때)

  - 업데이트 하는 방법은 `kubectl apply -f [DEPLOYMENT_NAME]` 을 통해서 가능함. 

  - 또 다른 방법은 `kubetctl set image deployment [DEPLOYMENT_NAME] [IMAGE] [IMAGE]:[TAG]` 를 통해서 가능하다. 

  - 또 다른 방법으로는 `kubectl edit deployment/[DEPLOYMENT_NAME]` 을 통해서 가능하다. 

- pod 를 업데이트 하는 과정은 이전에도 말했지만 new ReplicaSet 을 만듦으로써 가능하다. 

  - Replicaset 은 pod 의 수 뿐만 아니라 버전도 관리하므로. 

  - new ReplicaSet 의 pod 의 수는 늘리고 old Replicaset 의 pod 는 줄이고. 이걸 rolling update 라고 한다. (이 방식의 단점은 시간이 걸린다는 것. 그리고 트래픽을 컨트롤 할 수 없다는 점이다. old 로 갈 지 new 로 갈 지 알지 못하니.)


## Rolling update 

- rolling update 는 `max unavailable field` 와 `max surge field` 를 통해서 파드가 어떻게 업데이트 될 지 관리하는 방법이다. 

  - `max unavailable field` 는 roll out 중에 unavailable 할 pod 의 개수를 말한다. 예로 이 필드에 25%를 지정하면 4개 중 하나는 작동을 하지 않게 된다. 

  - `max surge field` 는 roll out  중에 New ReplicaSet 을 통해서 새롭게 만들 pod 의 최대 개수를 말한다. pod 의 개수가 4개인 상황에서 max surge field 에 25%를 지정하면 pod 가 5개가 된다.

  - 예시로 보자. `desired pods = 10` 이고 `max unavailable = 10%` 그리고 `max surge = 5` 라고 하고 roll out 을 한다고 해보자. 

    - old pods 가 10 개 있었고 새로운 파드가 5개 생긴다. 총 15 개가 되고 여기서 desired pod 는 10개니까 5개가 사라져야한다. 이건 old pods 에서 사라진다. 그리고 max unavailable 이 10% 니까 이건 desired pods 에서 계산되야 한다. 그러므로 파드가 하나 더 old 에서 사라져야한다. 즉 old = 4 new = 5 개가 됨. 

    - 그리고 이 과정이 한번 더 일어나면서 다시 new ReplicaSet 에서 pod 가 5개 생긴다. 그리고 10 개를 맞추기 위해서 old 에서 나머지 개수가 제거됨. 

- rolling update 에서 옵션으로 `min ready seconds` 와 `progress deadline seconds` 가 있음. 

 - `min ready seconds` 는 파드가 사용가능할 수준이 될때까지 기다리는 시간으로 기본 값은 0초이다. 즉 pod 가 준비되는 즉시 사용할 수 있다. 

 - `progress deadline seconds` 는 deployment 가 실패했을 때 report 할 때까지 기다리는 시간을 말한다. 
