# Canary Deployments 

- 카나리 배포 전략은 또 다른 배포 전략이다. 
- 일부 pod 만 새로운 버전으로 배포해놓고 정상적으로 작동하는지 확인하면 이후에 roll out 하는 방식. 
  - 예시로는 새로운 버전의 pod 에만 traffic 의 10% 정도 보내놓고 확인하는 식
  - 이 방식은 리소스 사용량을 적게 쓰면서 새로운 버전을 출시할 수 있다는 것. 일부 pod 만 출시한다는 것으로 이점을 얻을 수 있다. 
- blue-green 전략은 label 에 있는 version 으로 구분한 반면에 여기서는 그렇게 할 필요는 없다.
  - 여기서는 새로운 버전의 deployment 를 출시하고 replica 수를 조금 늘려서 확인하는 것. 확인은 모니터링과 performance 이슈가 있는지 체크. 
  - 그리고 문제가 있다면 롤백하고 문제가 없다면 새로운 버전의 앱은 스케일을 올리고, 오래된 버전의 앱은 스케일을 다운하는 식으로 진행. 
  - canary 배포 같은 경우는 istio 와 같은 트래픽을 분산시키는 툴이 있어야 한다. 
- 이외에도 배포 전략으로는 A/B Testing 같은게 있다.
  - 이런 전략들을 사용할 땐 session affinity 를 알고 필요하다면 적절하게 사용해야한다.
  - 일반적으로 service 는 같은 client 라도 같은 pod 에 요청을 보내지 않는다. 그냥 상황이 되는 아무 pod 에게나 보냄. 근데 앱의 기능이 많이 달라졌다면, pod 별로   
  기능이 많이 다르다면 같은 pod 에게 요청을 받도록 하는게 나을 수 있다. 그게 session affinity 이다. 여기에 clientIP 를 지정하면 가능해짐.  
  - 주로 A/B Testing 을 할 땐 browser version, user agent, geolocation, operation system 등과 관련해서 구별시킬 수 있다.
  - A/B Testing 은 유저의 행동을 관측하고 이를 바탕으로 비즈니스적 결정을 하는데 도움을 줄 수 있다.
  - Shadow testing 이라는 것도 있는데 이건 새로운 버전을 숨기고 유저의 요청에 대해 replay 하는 식으로 테스트를 한다.
    - 테스트 할 때 사이드 이펙트를 만들어내지 않는다는 장점이 있다.
    - `Diffy` 와 같은 툴들을 통해서 테스트하는게 가능하다. 이런 테스트를 통해서 error 나 performance 를 어플리케이션 버전에 따라서 비교하는게 가능하다. 
- roll out 은 stability 와 performance requirement 에 맞아야만 가능하다.
  - `kubectl rollout undo deployment [DEPLOYMENT_NAME]` 을 쓰면 deployment 를 revert 시킨다. 이전의 deployment 로.
  - `kubectl rollout undo deployment [DEPLOYMENT_NAME} --to-revision=2` 같은 걸 통해서 revision 레벨로 돌아가는 것도 가능하다.
  - `kubectl rollout history deployment [DEPLOYMENT_NAME]` 을 통해서 revision history 보는 것도 가능하다.
    - `kubectl rollout history deployment [DEPLOYMENT_NAME] --revision=2` 를 통해서 구체적인 히스토리를 볼 수 있다.  


![](./images/choosing%20the%20right%20startegy.png)

## Reviews

- Canary Deployment 는 stability 만 테스트 해볼 때 좋고, Shadow Testing 은 performance 요구사항까지 볼때. 
- 여러가지 배포 전략을 쓸 수 있어야겠다. 
   
  