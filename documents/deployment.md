# Deployment 

- 다 아는 내용은 뺴고. (pod 의 개수를 조절하거나, 배포를 할 때 어떻게 하는지. (새로운 Replicaset 을 만듦.))

- Deployment 는 stateless application 에 적합함. 
  - 즉 application data 나 state 를 cluster 의 persistent storage 에 저장하지 않는 어플리케이션을 말함. 

    - 이걸 하는 이유는 scalable 떄문. 
    - 대표적인 예는 front-end 

- Deployment 는 3가지 pod 의 라이프 사이클을 관리한다. 

  - Progressing State: Task 가 수행되는 State. 여기서 Task 는 ReplicaSet 을 만들거나 ReplicaSet 을 Scaling Up or Scaling Down 하는 걸 말한다. 

  - Complete State: ReplicaSet 을 최신버전으로 반영한 것. 즉 old ReplicaSet 은 작동하지 않는다.

  - Failed State: New ReplicaSet 의 생성이 완료되지 않을 때 발생함. 주로 쿠버네티스에서 Pod 의 image 를 불러오지 못할 때 pod 에 할당할 충분한 리소스가 없을 때 해당 command 를 실행시키는 user 의 permission 이 부족할 때 발생한다. 

- 새 버전을 출시하는 rollout 의 경우에는 small fix 만 적용되도록 관리하는게 좋다. 운영 복잡성을 줄이기 위해서, 어떤 버전으로 롤백되도록 할 지 명확하게 하기 위해서

- 그리고 yaml 파일의 경우에도 source code repository 에서 관리하도록 하는게 좋다. 

## Services and Scaling

- `kubectl autoscale deployment [DEPLOYMENT_NAME] --min=5 --max=15 --cpu-percernt=75`

  - auto scale 을 적용하는 것도 가능하다. 
  - 수평적 오토 스케일링은 갑작스러운 리소스 사용 증가에 적합하다. (CPU 증가나 requests per seconds 에 따라서.)

    - 상태를 가지지 않는 어플리케이션에 적합하다. 

  - 수직적 오토 스케일링은 시간에 따른 리소스 사용 증가에 적합하다. 또는 메모리에 기반해서. (시간 경과에 따른 리소스 요구사항을 보고 알맞은 사이즈로 오토 스케일링을 하는 것.)

    - 엡이 적은 리소스로만 운영된다면 throttled or out-of-memory-error 가 발생할 수 있다.

      - throttled 된다면 HPA (Horizontal pod autoscaler) 로 대응하면 되지 않겠냐 라고 물을 수 있는데 상태를 가진 어플리케이션이라면 수직적 스케일링을 해야할 것. 

    - 반대로 많은 리소스로 운영된다면 비용 문제가 발생할 것.  

    - 적합한 파드의 리소스 요구사항을 모를 때도 사용이 가능하다. 

- Horizontal pod autoscaler 는 cool down 과 delay 기능을 제공해준다. (scale down 이 되기전에 대기 시간을 주는 것.)
