# Controlling pod placement 

- 특정 pod 를 특정 조건에 해당하는 노드에 배치시키고 싶은 욕구가 있을 수 있다. 
  - 예로 디스크 타입이 SSD 인 노드에만 배치한다던지. 
  - 일반적으로 쿠버네티스는 모든 노드에 파드를 배치한다 실패의 위험을 줄이기 위해서. multi-zone 을 가졌다면 여러 zone 에다가 배포하기도 할 거임.
  - 아니면 AMD or Intel 64-bit 에 가진 노드에만 배치한다던지.
- 여기서는 pod 를 노드에 배치시키는 여러가지 방법을 배워본다.
  - GKE 에서는 Node pool 을 쓰는 걸 굉장히 권장한다. 결국 pod 를 노드에 배치시키는 건 해당 pod 가 올바른 하드웨어를 쓰는 것이 중요하기 떄문에. 
    - 이건 pod-pod 간의 관계를 신경쓰는 건 아니지만. 
    - nodeSelector 를 통해서 GKE 노드풀에 있는 노드를 선택하는 것 가능하다. 
      - `nodeSelector.cloud.google.com/gke-noodpool: [NODE_NAME]`
      - 노드풀에 있는 노드는 자동으로 노드 풀에 있는 네임인 label 이 붙는다.
    - nodeSelector 를 쓰는게 가장 간단한 방법. 그리고 그것보다 표현이 풍부한 방법이 Affinity 와 Anti-Affinity. (operator 가 풍부하고, pod 간의 관계도 명시하는게 가능.)
      

## Node Selector 

```yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx 
    imagePullPolicy: IfNotPresent
  nodeSelector:
    diskType: ssd
```

```yaml
apiVersion: v1
kind: Node
metadata: 
  name: node1
  labels: 
    diskType: ssd
```

- nodeSelector 를 pod spec 에 정의함으로써 특정 노드에 배치할 조건을 명시할 수 있다. 이 nodeSelector 와 node 에 있는 label 과 매칭되야한다. 
- 중간에 node 에 있는 label 을 바꾼다고 이미 실행중인 pod 에 영향을 주지는 않는다. 스케쥴링 할 때만 영향줌.

## Node Affinity and Anti-Affinity 

- Node Affinity 는 Node Selector 와 마찬가지로 pod 를 스케줄링 할 때 어떤 노드에 배치되어야 하는가를 제한하는 용도로 쓰인다. 
- 근데 Affinity 가 Selector 보다 더 표현이 풍부하다. 노드에 대한 조건 뿐 아니라, 노드 안에 실행되고 있는 pod 에 대한 조건까지 쓸 수 있어서.
  - 여러 조건들도 넣을 수 있어서 더 풍부함. 
- Node Affinity 와 Anti-Affinity 는 파드가 launching 되는 걸 막아주지만 Node Selector 는 pod 가 스케줄링 되는걸 막아준다.

### 예시로 보자. 

```yaml
apiVersion: v1
kind: Pod 
metadata: 
  name: with-node-affinity 
spec: 
  affinity: 
    nodeAffirnity: 
      requiredDuringSchedulingIgnoredDuringExecution: 
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator-type
            operator: In
            values:
            - gpu
            - tpu
    podAffinity: 
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100 
        podAffirnityTerm: 
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - webserver
        topologyKey: failure-domain.beta.kubernetes.io/zone
```

- affinity 와 anti-affinity 는 `requiredDuringSchedulingIgnoredDuringExecution` 와 `preferredDuringSchedulingIgnoredDuringExecution` rule 로 완성된다.
  - affinity 가 preferred 를 말하고 anti-affinity 가 required 를 말하는듯.
  - 이것도 node-label 에 기반해서 설정하는 것임. (그래서 node-selector 와 유사하다.) 
  - `requiredDuringSchedulingIgnoredDuringExecution` 는 조건에 맞지 않으면 파드가 스케줄링 되지 않음.
  - `preferredDuringSchedulingIgnoredDuringExecution` 는 조건에 맞는 노드를 찾는다. 근데 찾지 못하더라도 스케줄링 안되는 건 아님.
    - 이건 weight 라는 스코어 값을 통해서 우선순위를 매김. 이 값이 가장 높은 노드에 스케줄링 됨.
  - `IgnoredDuringExecution` 이 뜻은 중간에 label 이 바뀌더라도 실행되는데는 영향이 없다는 뜻임.
- 하나의 `nodeSelectorTerms` 에 여러개의 `matchExpressions` 를 넣는 것도 가능하다. 이 경우에는 모든 조건에 맞아야 실행이 됨. (requiredDuringSchedulingIgnoredDuringExecution 의 경우.)
- `nodeSelectorTerms` 를 여러개  쓰는 경우 하나의 조건에만 만족해도 스케줄링이 될 수 있다.
- 여기서는 `operator` 가 `In` 으로 쓰였지만 여러개가 있다. `NotIn, Exists, DoesNotExist, Gt, Lt`
- affinity 에는 pod-level 에서 지정할 수 있도록 하는 podAffinity 도 있다. 
  - node 에 배포되어있는 pod label 을 조건으로 해당 노드에 배치될 지 말지를 결정할 수 있다.
- topologyKey 를 통해서 노드 레벨보다 더 고레벨로 설정하는게 가능하다. 
  - zone 이나 region 을 지정하는게 가능함. 
  - topologykey 는 node label 의 key 로 통하고 topology 는 <key, value> pair 와 같다.
  - 같은 key,value 는 같은 topology 에 속한다. 

## Pod Placement Example

- 실제 예시로 보면서 배우는 것.
- webserver 인 pod 는 서로 같은 zone 에서 실행되지 않도록 하고, 캐시 pod 와 같은 노드에 배치되도록 선호하는 조건을 단다. 
- 캐시 pod 는 서로 같은 zone 에서 실행되지 않도록 하고, webserver pod 와 같은 노드에 배치되는 걸 선호하도록 한다.

## Taints and Toleration 

- (taint 뜻은 오염시키다. toleration 은 tolerance 와 같고, 관용이란 뜻이다.)
- taint 는 노드에 적용시켜서 pod 를 `NoSchedule` or `NoExecute` 설정을 해서 pod 를 노드에 배치시키지 않도록 한다. 
- 이런 taint 에 대응할 수 있도록 하는게 pod 쪽에 `toleration` 을 거는 것. 
- taint 와 toleration 을 앎으로써 노드에 pod 를 배치시키는 유연함을 가질 수 있다. 
- taint 의 effect 들에 대해서 정리해보면 다음과 같다. 
  - NoSchedule 
    - hard limit 로 pod 의 toleration 에 있지 않으면 pod 는 해당 노드에 schedule 되지 않는다. 
  - PreferNoSchedule 
    - soft limit 로 pod 의 toleration 에 있지 않으면 노드에 배치할려고 하지 않지만 배치될 수 있다.
  - NoExecute 
    - pod 의 최소한 하나의 toleration 에서 taint 와 같은게 없다면 실행 중인 모든 Pod 를 제거한다.   

### taint 와 toleration 적용 

```
$ kubectl taint nodes node1 key1=value1:NoSchedule
```

````yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
````

- taint 는 kubectl 을 통해서 적용하는게 가능하다.
- 이런 taint 와 pod.tolerations 가 매칭이 되면 해당 pod 는 노드에 스케쥴링 되는게 가능하다.
  - 여기서 operator 는 Equals 이기 떄문에 key, value, effect 가 모두 taint 와 같아야한다.
  - operator 가 Exists 이라면 key 와 effect 만 같으면 된다. 