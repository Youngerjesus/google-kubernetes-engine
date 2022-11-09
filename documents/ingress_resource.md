# Ingress Resource 

- ingress 를 통해서 트래픽을 처리할 수 있다.
- Ingress 는 service 위에 있는 레이어로 service 들을 위한 service 이다. (그렇다고 service 종류는 아님.) 
  - Ingress 밑에 여러개의 service 가 있는것.
- Ingress 를 통해서 하나의 public IP 를 들어내고 여기에 들어오는 HTTP, HTTPS 트래픽에 대한 로드밸런싱을 Service 에 제공해준다. 
- GKE 에서 Ingress 는 Google Cloud Load Balancer 와 Service 가 결합된 것이라고 생각하면 된다.

#### Example 

````yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata: 
  name: my-ingress
spec:
  backend:
    serviceName: default 
    servicePort: 80 
  rules:
  - host: demo.example.com
    http: 
      paths: 
      - path: /demo1examplepath
          backend:
            serviceName: demo
            servicePort: 80
  - host: lab.user.com
    http: 
      paths: 
      - path: /labpath
          backend: 
            serviceName: lab1
            servicePort: 80
````   

- ingress 에는 여러개의 host-service 를 매핑하는 것이 가능하다. 
- 그리고 ingress 에는 기본적으로 연결될 서비스를 지정하는 것도 가능하다. (이걸 지정하지 않으면 받을 수 없는 요청에 대래서는 404 가 호출된다.)