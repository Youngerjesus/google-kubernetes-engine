# Kubernetes concepts

## Overview 

- K8s 가 어떻게 동작하는지 이해하기 위해서는 2가지 개념을 알아야한다. 
  - 첫 번째는 Kubernetes 는 모든 걸 object 로 관리한다는 것. 그래서 이런 object 들을 볼 수 있고 attributes 와 state 도 볼 수 있다. 
  - 두 번째는 declarative management 에 대해서 알아야한다. 내가 원하는 object 의 상태를 기술하면 쿠버네티스가 그걸 유지해준다.
    - 이걸 유지하게 하는 방법은 `watch loop` 때문이다. 
- K8s Object 에서는 두 가지 중요한 요소가 있다. 
  - 내가 만들고자 하는 object 에 대해서 spec 을 기술하면 된다든 것. 
    - object 의 상태는 k8s control plane 에서 제공한다.
    - control plane 에서 감시하고 고치는 노력을 한다.
  - 그리고 각각의 오브젝트에는 오브젝트 타입을 기술하는 `kind` 라는게 있다.  
- K8s 에서 배포되는 가장 작은 모델은 pod 라는 것. 
  - pod 안에 하나 이상의 container 가 있다.
  - pod 안에 여러 컨테이너가 있다면 이는 강하게 결합된 것으로, Storage 와 Network 를 공유한다.
    - 그래서 하나의 컨테이너가 같은 파드에 있는 컨테이너와 통신할려면 localhost 와 통신하는 것과 같다. 
  - 쿠버네티스는 pod 에 고유의 IP 를 부여한다.  