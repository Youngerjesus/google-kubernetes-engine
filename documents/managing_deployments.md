# Managing Deployments

- deployment 를 edit 하는 순간 자동으로 roll out 된다. 
- 이렇게 되면 roll out 버전이 많아져서 문제가 생길 떄 어떤 버전으로 roll back 되야하는지 알기 어렵다. 
- 그래서 작은 버전의 fix 를 모아서 roll out 하도록 잠시 roll out 을 멈출 수 있는 기능으로 `kubectl rollout pause deployment [DEPLOYMENT_NAME]` 이 있다. 
- 이걸하고 `kubectl rollout resume deployment [DEPLOYMENT_NAME]` 을 하면 한번에 roll out 하는게 가능하다.
- roll out 이 잘되었는지 볼려면 `kubectl rollout status deployment [DEPLOYMENT_NAME]` 으로도 가능하다.
- deployment 를 지울려면 `kubectl delete deployment [DEPLOYMENT_NAME]` 을 통해서 가능하다. 