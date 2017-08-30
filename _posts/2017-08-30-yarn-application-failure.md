---
layout: post
title:  "얀 어플리케이션 실패 - '각 프로그램들은 어떻게 동작할까?'"
date:   2017-08-30 16:40:22 +0900
categories: psyoblade update
---

## 참고사항
> 아래의 글이 괜찮은 것 같아 작성하였으나, 다 읽을 즈음에는 이 글이 2014년 7월 글이고, Out-dated 되었다는 걸 알게 되었습니다. 다만, 기본적인 동작 방식은 유사하기 때문에 참고할 만한 글이라 그대로 저장해 둡니다.



## [Failures in YARN](http://sungsoo.github.io/2014/04/07/failures-in-yarn.html)
> YARN 환경에서 *task, application master, node manager and resource manager* 등의 다양한 컴포넌트 장애 상황에서 YARN은 어떻게 동작하는지 살펴보겠습니다.

### 1. task 실패
> Failure of the running task is similar to the classic case. Runtime exceptions and sudden exits of the JVM are propagated back to the application master and the task attempt is marked as failed.
* 실행 중인 작업의 실패는 고전적인 케이스이며, 대부분 런타임 오류나 JVM의 예기치 않은 종료인데 이 경우 aplication master로 전달되고, 실패로 마킹됩니다. 

> Likewise, hanging tasks are noticed by the application master by the absence of a ping over the umbilical channel (the timeout is set by mapreduce.task.timeout), and again the task attempt is marked as failed. 
* 행에 걸린 작업들은 application master의 umbilical channel을 통해 ping이 없는 경우 인지되는데 (mapreduce.task.timeout), 이 또한 실패로 마킹됩니다.

> The configuration properties for determining when a task is considered to be failed are the same as the classic case: a task is marked as failed after four attempts (set by mapreduce.map.maxattempts for map tasks and mapreduce.reduce.maxattempts for reducer tasks). A job will be failed if more than mapreduce.map.failures.maxpercent percent of the map tasks in the job fail, or more than mapreduce.reduce.failures.maxpercent percent of the reduce tasks fail.
* 하둡 설정파일에 의해 이러한 고전적인 실패의 4회 시도가 감지되면, mapreduce.{map|reduce}.failures.maxpercent 비율을 넘는 경우 job 실패로 간주하고 종료됩니다.


### 2. application master 실패
> Just like MapReduce tasks are given several attempts to succeed (in the face of hardware or network failures) applications in YARN are tried multiple times in the event of failure. By default, applications are marked as failed if they fail once, but this can be increased by setting the property yarn.resourcemanager.am.max-retries.
* MapReduce 작업들과 같이 몇 번 재시도 하는 것 처럼, YARN의 application들도 task가 실패하는 경우 수차례 재시도 할 수 있습니다만, 기본값은 한 번 실패시에 실패로 간주됩니다. 다만, yarn.resourcemanager.am.max-retries 값으로 변경할 수 있습니다.

> An application master sends periodic heartbeats to the resource manager, and in the event of application master failure, the resource manager will detect the failure and start a new instance of the master running in a new container (managed by a node manager). In the case of the MapReduce application master, it can recover the state of the tasks that had already been run by the (failed) application so they don’t have to be rerun. By default, recovery is not enabled, so failed application masters will not rerun all their tasks, but you can turn it on by setting yarn.app.mapreduce.am.job.recovery.enable to true.
* application master는 주기적으로 resource manager에 핫빗을 날리고, application master 실패가 인지됩니다. resource manager는 실패를 인지하면 새로운 container에 새로운 application master 인스턴스를 띄우게 됩니다. MapReduce application master의 경우 이전 작업들의 상태를 복구할 수 있지만, 굳이 다시 실행할 필요가 없기 때문에 복구의 기본값은 disabled 되어 있습니다. yarn.app.mapreduce.am.job.recovery.enable 값을 true 변경하면 됩니다. 

> The client polls the application master for progress reports, so if its application master fails the client needs to locate the new instance. During job initialization the client asks the resource manager for the application master’s address, and then caches it, so it doesn’t overload the the resource manager with a request every time it needs to poll the application master. If the application master fails, however, the client will experience a timeout when it issues a status update, at which point the client will go back to the resource manager to ask for the new application master’s address.
* 클라이언트는 application master에 현재 상태를 확인하기 위해 계속 폴링하게 되는데, application master 실패 시에는 클라이언트는 새로운 인스턴스를 찾아야만합니다. job 초기 화시에 client는 resource manager에 application master 의 address를 물어보고 캐싱하게 되는데, application master 실패 시에 timeout이 발생하고, 상태를 업데이트 한 후 resource manager에 다시 주소를 물어 새로운 application master의 주소를 알아낼 수 있습니다.


### node manager 실패
> If a node manager fails, then it will stop sending heartbeats to the resource manager, and the node manager will be removed from the resource manager’s pool of available nodes. The property yarn.resourcemanager.nm.liveness-monitor.expiry-intervalms, which defaults to 600000 (10 minutes), determines the minimum time the resource manager waits before considering a node manager that has sent no heartbeat in that time as failed.
* node manager 실패 시에 resource manager로 전송되는 핫빗이 멈추게되고, node manager는 resource manager의 pool로부터 분리되게 됩니다. yarn.resourcemanager.nm.liveness-monitor.expiry-intervalms 값으로 설정되고 기본값은 10분(600000)입니다. 이 값이 resource manager가 node manager가 살아있다는 핫빗을 기다리는 최대 시간입니다.

> Any task or application master running on the failed node manager will be recovered using the mechanisms described in the previous two sections.
* 실패한 node manager 상에서 수행되는 어떤한 task 혹은 application master 는 두 번째 섹션에서 기술한 매커님즘에 의해 복구가 가능합니다.

> Node managers may be blacklisted if the number of failures for the application is high. Blacklisting is done by the application master, and for MapReduce the application master will try to reschedule tasks on different nodes if more than three tasks fail on a node manager. The threshold may be set with mapreduce.job.maxtaskfailures.per.tracker.
* node managers 는 application이 많이 실패한 노드를 블랙리스트를 관리하고 있으며, application master에 의해 블랙리스트는 결정됩니다. 그리고 MapReduce application master는 회 이상 task실패가 있었던 node manager의 경우는 다른 노드를 찾도록 스케줄링을 조절합니다. 이 임계치는 mapreduce.job.maxtaskfaiures.per.tracker 에 의해 조정됩니다.

### resource manager 실패
> Failure of the resource manager is serious, since without it neither jobs nor task containers can be launched. The resource manager was designed from the outset to be able to recover from crashes, by using a checkpointing mechanism to save its state to persistent storage, although at the time of writing the latest release did not have a complete implementation.
* resouce manager의 실패는 어떠한 job, task, container도 실행할 수 업식 때문에 가장 심각한 문제입니다.  resource manager의 경우 외부 persistent storage에 checkpoint를 저장하는 매커니즘으로 crash로부터 복구가 가능하도록 설계되어 있습니다. 이 글을 작성하는 최근 릴리스에는 완전한 구현은 되어 있지 않습니다. (이 글을 작성하는 시점은 2014년이고, 현재 hadoop 2.7.3 버전은 이러한 문제가 모두 해결 되었습니다.)

> After a crash, a new resource manager instance is brought up (by an adminstrator) and it recovers from the saved state. The state consists of the node managers in the system as well as the running applications. (Note that tasks are not part of the resource manager’s state, since they are managed by the application. Thus the amount of state to be stored is much more managable than that of the jobtracker.)
* crash 후에는 새로운 resource manager 인스턴스가 관리자에 의해 실행되어 져야하고, 저장된 상태로부터 복구가 이루어집니다. 

> The storage used by the reource manager is configurable via the yarn.resourceman ager.store.class property. The default is org.apache.hadoop.yarn.server.resource manager.recovery.MemStore, which keeps the store in memory, and is therefore not highly-available. However, there is a ZooKeeper-based store in the works that will support reliable recovery from resource manager failures in the future.
* resousrce maanager에 의해 사용되는 저장소는 yarn.resourcemanager.store.class.property 값으로 지정되고 기본은 MemStore입니다.




## [MapReduce on YARN Job Execution](https://www.slideshare.net/martyhall/hadoop-tutorial-mapreduce-part-6-job-execution-on-yarn)

