# Jobs and CronJobs 

- Job 은 하나 이상의 pod 를 만들어서 reliable 한 Task 를 수행하는데 사용할 수 있다.
  - (tracking 하는 pod 를 만든다는건가, Task 를 수행하는 pod 에서 report 를 보낸다는 건가)
  - Task 가 완료되었다면 Pod 는 종료되고 Job 에게 report 를 보낸다.  
  - Pod 가 어떠한 이유로든 종료가 된다면 다시 재시작 하지는 않지만 Job 은 이러한 유형의 에러를 핸들링할 수 있다.
    - (어떻게 핸들린한다는거지? Job Controller  가 Job 을 Node 에서 실행시키도록 한다. 그리고 Node 에서 실패해서 Task 가 완료되지 않는다면 Job Controller 가 다른 노드에서 pod 를 시작하는거임.)
    - report 는 Job Controller 에게 보내겠네.  
    - Job Controller 가 계속해서 Task 가 완료되었는지 모니터링하고.
- Job 을 정의하는 방법은 2가지가 있다.
  - non-parallel
    - Task 를 수행하는 pod 는 오로지 하나. 
  - parallel 
  
#### Job Yaml

````yaml
apiVersion: batch/v1
kind: Job 
metadata:
  name: my-app-job
spec:
  completions: 3
  parallelism: 2 # parallelism 을 사용하는 경우에 필요.
  template:
    :spec:
````

- completions count 는 수행되야할 task 를 말한다. 
- completions count 가 정의될 때 parallel job 는 parallelism value 를 가질 수 있고 이를 통해서 multiple pod 를 동시에 실행하는게 가능하다.

#### Job non-parallel

````yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi 
spec: 
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wie", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4 
```` 

- restartPolicy 를 통해서 pod 가 task 를 수행하는 도중 실패했을 경우 어떻게 대응할지를 정할 수 있다.
  - 여기서 never 라고 지정한 경우 하나의 pod 나 컨테이너가 실패하면 전체의 pod 가 실패하고 다시 시작한다.
  - 이 값 말고 `OnFailure` 도 있는데 이건 해당 노드에서 실패한 경우 restart 할 때도 그 노드에서 시작한다. 
- backOffLimit 를 통해서 restart count 를 지정하는 것도 가능하다.   
  - 기본값은 6이다. 
  - 다시 생성될 때 delay 를 가지고 생성된다. 지수적으로 증가하는 딜레이. 10초, 20초, 40초... 최대 6분. 

### Parallel Jobs 

- 같은 Task 를 수행하는데 Multiple Pod 를 통해서 해결하는 것. 
- Parallel Job 을 설정하는 방법은 `spec.parallelism` 을 1보다 높게 설정하는 것으로 가능.
- Parallel Job 은 크게 두 가지가 있다. (둘 다 병렬로 )
  - fixed Task 를 처리하는 것. 
  - a work queue 를 처리하는 것. 철
- fixed Task 
  - `parallelism` 에 적힌 수 만큼 pod 를 생성한다. 그리고 pod 가 `completions` 개수만큼 처리할떄가지 진행. 
  - `parallelism` 으로 생성된 pod 가 task 를 끝냈는데 `completions` 보다 작다면 job Controller 가 새로운 pod 를 또 생성하는 방식.
  - 만약 현재 시점에 남은 Task 의 수가 parallism 보다 작고 남아 있는 pod 로 해결할 수 있다면 이제 새로운 pod 를 새엇ㅇ하지 않는다.
- work queue
  - queue 에 item 이 없을 떄까지 작동한다. 
  - `parallelism` 만 설정하고 `completions` 는 설정하지 않는 경우에 사용가능하다.
  - 이 경우에 job controller 는 queue 가 비어있고 pod 가 종료되어야 한다는 걸 모른다. 오로지 pod 에게 의존적.
- job 은 `kubectl describe job [JOB_NAME]` 을 통해서 보거나 `kubectl get pod -L [job-name=JOB_NAME]` 을 통해서 가능하다.
  - job-name 은 label selector 에 의해서 필터링 하는것.
- `activeDeadlineSeconds` 를 통해서 잡의 생존 시간을 지정하는 것도 가능하다.
- job pod 를 유지하면서 job 을 제거할려면 `kubectl delete job [JOB_NAME] --cascade false` 로 설정하면 된다.     

### Cron Jobs 

- 반복되서 실행되야 하는 job 의 경우 cron syntax 를 이용한 job 을 만들 수 있다. 이건 date 와 time 으로 만듦.
- cron job 을 스케쥴 걸었는데 실행되지 않고 이 카운트가 100이 넘는다면 에러 로그가 찍히고 job 은 이제 실행되지 않는다. 이건 마지막 job 스케쥴 이후부터 체크하는 거임.
  - 마지막 job 스케쥴 이후부터 걸었기 떄문에 failed count 가 누적되는 걸 막는다.  
  - 이 기간은 `startingDeadlineSeconds` 값으로부터 조절하는 것도 가능하다. 이 값은 `startingDeadlineSeconds` 부터 `now` 까지 실패한 카운트의 개수를 재는 window time 이다.
- cron job 은 job 의 스케쥴 타임과 task 를 끝내는 시간에 따라서 하나 이상의 cron job 이 동시에 실행되는 경우가 생길 수 있다. 이걸 조절할려면 `concurrencyPolicy` 를 설정해야한다.
 - 이 값은 `Allow, Forbid, Replace` 가 있다.  
 - `Forbid` 의 경우 하나의 잡이 끝나지 않았는데 스케쥴 타임이 온다면 실행하지 않는다. 그리고 failed count 를 증가시킨다. 
 - `Replace` 는 새로운 잡으로 대체된다. 
- `suspend` 속성을 true 로 설정한다면 이제부터 모든 새로운 job 은 suspend 가 가능하다. 그리고 이 값은 missed job 으로 간주됨.
  - 아 suspend, resume 을 위해서 필요한 설정으로, 클러스터에서 자원이 한정되어 있어서 지금 실행하기에 적합하지 않거나, 우선순위가 밀리거나 하는 경우 suspend 하기 위해서 쓰는 설정이다.
    - job 을 삭제하는 건 좋은 솔루션이 아니니까. 
- `successfulJobsHistoryLimit` 와 `failedJobsHistoryLimit` 은 history 개수를 말한다. 
  

### cron foramt 

```
0 * * * *: Every hour

*/15 * * * *: Every 15 minutes

0 */6 * * *: Every 6 hours 

0 20 * * 1-5: Every week Mon-Fri at 8:00 pm 

0 0 1 */2 *: At 00:00 on day 1 of every other month
``` 

- 분 시간 일 월 요일 로 해당됨.
- 분은 0-59 
- 시간은 0-23
- 일은 1-31
- 월은 1-12
- 요일은 0-6 
- `/` 은 증분을 말한다. `*/2` 를 시간에 썼다면 2시간마다 증분을 말한다.
  - `*/5 * * * *` 는 5분마다 실행되는 걸 말한다면 `5 * * * *` 는 매 시간마다 5분에 실행되는 걸 말함.
  
### Cron Schedule 

````yaml
apiVersion: batch/v1
kind: CronJob 
metadata:
  name: my-app-job
spec:
  schedule: "*/1 * * * *" 
  startingDeadlineSeconds: 3600
  concurrencyPolicy: Forbid 
  suspend: true 
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobtemplate:
    spec: 
      template:
        spec:
````  