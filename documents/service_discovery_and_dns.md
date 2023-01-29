# Service Discovery and DNS 

- GKE 가 service discovery 와 cluster dns 를 어떻게 구현헀는지 알려주는 내용.
- K8s 에서는 serviceName to serviceIP 가 매핑되는 형태를 이용해서 service discovery 를 해결한다.
  - serviceName 예시) `my-svc.my-namespace.svc.cluster-domain.example.`
- GKE 에서는 serviceName 과 externalName 을 resolve 하기 위해서 다음과 같은 cluster DNS 를 제공한다. 
  - `kube-dns`
    - 모든 GKE 클러스터에 기본적으로 들어있는 것.
    - deployment 로 관리됨.
  - `cloud-dns`
    - Google 에 의해서 관리되어지는 DNS Resolver
- cluster domain 을 보고 cluster domain 이라면 cluster dns (=core dns) 에 호출하고 아니라면 external dns server 로 호출한다.

## Service discovery outside a single cluster
