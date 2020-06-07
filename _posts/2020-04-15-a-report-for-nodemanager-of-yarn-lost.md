---
layout: post
title: "NodeManager丢失调查报告"
subtitle: "The report for nodemanager of yarn lost"
author: "qingshan"
header-img: "img/post-bg-kuaidi.jpg"
header-mask: 0.4
tags:

- 工作
- Yarn
- Hadoop
---

# 一、前言
2020年04月，发现NodeManager节点数据总数异常。经过对比DataNode列表，并排除故障机、维修机后，仍然有21个节点的NodeManager丢失。

通过查看以上节点的进程情况，发现节点中的NodeManager进程存在，但没有在正常工作。通过yarn-daemon.sh stop nodemanager 命令无法正常结束进程。查看日志，是LOCK文件没有被清理
```bash
2020-04-14 15:43:01,437 INFO [main] org.apache.hadoop.service.AbstractService: Service org.apache.hadoop.yarn.server.nodemanager.recovery.NMLeveldbStateStoreService failed in state INITED; cause: org.fusesource.leveldbjni.internal.NativeDB$DBException: IO error: lock /home/yarn/nm_recovery/yarn-nm-state/LOCK: Resource temporarily unavailable
```

# 二、调查经过
在上述异常节点中，抽取sd-hadoop-datanode-50-168主机上的日志文件（/home/vipshop/logs/yarn/yarn-yarn-nodemanager-sd-hadoop-datanode-50-168.idc.vip.com.log)查看，定位到日志异常前的可疑位置：

```bash
2019-12-16 18:36:23,543 WARN [Node Status Updater] org.apache.hadoop.yarn.server.nodemanager.NodeStatusUpdaterImpl: Node is out of sync with ResourceManager, hence resyncing.
2019-12-16 18:36:23,543 WARN [Node Status Updater] org.apache.hadoop.yarn.server.nodemanager.NodeStatusUpdaterImpl: Message from ResourceManager: Node not found resyncing sd-hadoop-datanode-50-168.idc.vip.com:45454
```

发现NodeManager接收到了ResourceManager的信息 Node not found resyncing，对比代码，发现相关代码逻辑如下：
```java

protected void startStatusUpdater() {
 
  statusUpdaterRunnable = new Runnable() {
    @Override
    @SuppressWarnings("unchecked")
    public void run() {
      int lastHeartBeatID = 0;
      while (!isStopped) {    <== 代码定位
        // Send heartbeat
        try {
          NodeHeartbeatResponse response = null;
          NodeStatus nodeStatus = getNodeStatus(lastHeartBeatID);
           
          NodeHeartbeatRequest request =
              NodeHeartbeatRequest.newInstance(nodeStatus,
                NodeStatusUpdaterImpl.this.context
                  .getContainerTokenSecretManager().getCurrentKey(),
                NodeStatusUpdaterImpl.this.context.getNMTokenSecretManager()
                  .getCurrentKey());
          response = resourceTracker.nodeHeartbeat(request);
          //get next heartbeat interval from response
          nextHeartBeatInterval = response.getNextHeartBeatInterval();
          updateMasterKeys(response);
 
          if (response.getNodeAction() == NodeAction.SHUTDOWN) {
            LOG
               .warn("Recieved SHUTDOWN signal from Resourcemanager as part of heartbeat," 
                  + " hence shutting down.");      
            LOG.warn("Message from ResourceManager: "   
                + response.getDiagnosticsMessage());   
            context.setDecommissioned(true);   
            dispatcher.getEventHandler().handle(   
                new NodeManagerEvent(NodeManagerEventType.SHUTDOWN));
            break;
          }
          if (response.getNodeAction() == NodeAction.RESYNC) {
代码定位 ==> LOG.warn("Node is out of sync with ResourceManager,"
                + " hence resyncing.");
            LOG.warn("Message from ResourceManager: "
                + response.getDiagnosticsMessage());
            // Invalidate the RMIdentifier while resync
            NodeStatusUpdaterImpl.this.rmIdentifier =
                ResourceManagerConstants.RM_INVALID_IDENTIFIER;
            dispatcher.getEventHandler().handle(
                new NodeManagerEvent(NodeManagerEventType.RESYNC));
            pendingCompletedContainers.clear();
            break;
          }
```

可以看到此时正常触发了 dispatcher.getEventHandler().handle(new NodeManagerEvent(NodeManagerEventType.RESYNC)) 这个RESYNC事件，日志记录准备RESYNC的过程如下：
```bash
2019-12-16 18:36:23,551 INFO [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: Containers still running on ON_NODEMANAGER_RESYNC : [container_1575307473787_4683518_01_000108, container_1575307473787_4683518_01_000109, container_1575307473787_4683518_01_000110, container_1575307473787_4683518_01_000111, container_1575307473787_4683518_01_000115, container_1575307473787_4683518_01_000116, container_1575307473787_4683518_01_000117, container_1575307473787_4683518_01_000118, container_1575307473787_4683518_01_000119, container_1575307473787_4683518_01_000120, container_1575307473787_4683518_01_000121, container_1575307473787_4683518_01_000122, container_1575307473787_4683518_01_000124, container_1575307473787_4683518_01_000125, container_1575307473787_4683518_01_000126, container_1575307473787_4683518_01_000127, container_1575307473787_4683518_01_000128, container_1575307473787_4683518_01_000129, container_1575307473787_4683518_01_000130, container_1575307473787_4683518_01_000131, container_1575307473787_4683518_01_000132, container_1575307473787_4683518_01_000133, container_1575307473787_4683518_01_000134, container_1575307473787_4683518_01_000135, container_1575307473787_14283729_01_000305, container_1575307473787_14283729_01_000306, container_1575307473787_14283729_01_000307, container_1575307473787_14283729_01_000308, container_1575307473787_14285364_01_006226, container_1575307473787_14285364_01_006227, container_1575307473787_14285364_01_006228, container_1575307473787_14285364_01_006229, container_1575307473787_14285364_01_006230, container_1575307473787_14285364_01_006231, container_1575307473787_14285364_01_006232, container_1575307473787_14285364_01_006233, container_1575307473787_14285364_01_006234, container_1575307473787_14285364_01_006235, container_1575307473787_14285364_01_006236, container_1575307473787_14285364_01_006237, container_1575307473787_14285364_01_006238, container_1575307473787_14285364_01_006239, container_1575307473787_14285364_01_006240, container_1575307473787_14285364_01_006241, container_1575307473787_14285364_01_006242, container_1575307473787_14285364_01_006243, container_1575307473787_14285364_01_006244, container_1575307473787_14285364_01_006245, container_1575307473787_14285364_01_006246, container_1575307473787_14285364_01_006247, container_1575307473787_14285364_01_006248, container_1575307473787_14285364_01_006249, container_1575307473787_14285364_01_006250, container_1575307473787_14285364_01_006251, container_1575307473787_14285364_01_006252, container_1575307473787_14285364_01_006253, container_1575307473787_14285364_01_006254, container_1575307473787_14285987_01_002667, container_1575307473787_14285987_01_002668, container_1575307473787_14285987_01_002669, container_1575307473787_14285987_01_002670, container_1575307473787_14285987_01_002671, container_1575307473787_14285987_01_002672, container_1575307473787_14285987_01_002673, container_1575307473787_14285987_01_002674, container_1575307473787_14285987_01_012249, container_1575307473787_14286954_01_000025, container_1575307473787_14286954_01_000026, container_1575307473787_14286954_01_000027, container_1575307473787_14286954_01_000028, container_1575307473787_14286954_01_000029, container_1575307473787_14286954_01_000030, container_1575307473787_14286954_01_000031, container_1575307473787_14286954_01_000032, container_1575307473787_14286954_01_000033, container_1575307473787_14286954_01_000034, container_1575307473787_14286954_01_000035, container_1575307473787_14286954_01_000036, container_1575307473787_14286954_01_000037, container_1575307473787_14286954_01_000038, container_1575307473787_14286954_01_000039, container_1575307473787_14286954_01_000040, container_1575307473787_14286954_01_000041, container_1575307473787_14286954_01_000042, container_1575307473787_14286954_01_000043, container_1575307473787_14286954_01_000044, container_1575307473787_14286954_01_000045, container_1575307473787_14286954_01_000046, container_1575307473787_14286954_01_000047, container_1575307473787_14286954_01_000048, container_1575307473787_14286954_01_000049, container_1575307473787_14286954_01_000050, container_1575307473787_14286954_01_000051, container_1575307473787_14286954_01_000052, container_1575307473787_14286954_01_000053, container_1575307473787_14286954_01_000054, container_1575307473787_14286954_01_000055, container_1575307473787_14286954_01_000056, container_1575307473787_14286954_01_000057, container_1575307473787_14286954_01_000058, container_1575307473787_14286954_01_000059, container_1575307473787_14286954_01_000060, container_1575307473787_14286954_01_000061, container_1575307473787_14286954_01_000062, container_1575307473787_14286954_01_000063, container_1575307473787_14286954_01_000064, container_1575307473787_14286954_01_000065, container_1575307473787_14288603_01_000001]
2019-12-16 18:36:23,551 INFO [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: Waiting for containers to be killed
```

可以看到，程序调用了cleanupContainersOnNMResync() 方法，列出了需要resync的container 列表，并通过handle处理向CMgrCompletedContainersEvent发送了ON_NODEMANAGER_RESYNC事件。接下来 CMgrCompletedContainersEvent接受到事件，并且又触发了ContainerManagerEventType.FINISH_CONTAINERS事件。
```java
public void cleanupContainersOnNMResync() {
  Map<ContainerId, Container> containers = context.getContainers();
  if (containers.isEmpty()) {
    return;
  }
  LOG.info("Containers still running on "
      + CMgrCompletedContainersEvent.Reason.ON_NODEMANAGER_RESYNC + " : "
      + containers.keySet());
 
  List<ContainerId> containerIds =
    new ArrayList<ContainerId>(containers.keySet());
 
  LOG.info("Waiting for containers to be killed");
 
  this.handle(new CMgrCompletedContainersEvent(containerIds,
    CMgrCompletedContainersEvent.Reason.ON_NODEMANAGER_RESYNC));  <== 代码定位
 
 
public CMgrCompletedContainersEvent(List<ContainerId> containersToCleanup,
                                    Reason reason) {
  super(ContainerManagerEventType.FINISH_CONTAINERS);    <== 代码定位
  this.containerToCleanup = containersToCleanup;
  this.reason = reason;
}
```

ContainerManagerEventType.FINISH_CONTAINERS逻辑如下，得知这里试图获取 Application Id，但是没有成功。
```java
@Override
public void handle(ContainerManagerEvent event) {
 // 省略无关代码若干
case FINISH_CONTAINERS:
  CMgrCompletedContainersEvent containersFinishedEvent =
      (CMgrCompletedContainersEvent) event;
  for (ContainerId containerId : containersFinishedEvent
      .getContainersToCleanup()) {
    ApplicationId appId =
            containerId.getApplicationAttemptId().getApplicationId();
    Application app = this.context.getApplications().get(appId);    <== 代码定位
    if (app == null){      <== 代码定位
      LOG.warn("couldn't find app " + appId + "while processing"    <== 代码定位
              + " FINISH_CONTAINERS event");
      continue;
    }
    Container container = app.getContainers().get(containerId);
    if(container == null ){    <== 代码定位
      LOG.warn("couldn't find container "+ containerId   <== 代码定位
              + " while processing FINISH_CONTAINERS event");
      continue;
    }
    if (container.isRecovering()){   
      LOG.info("drop FINISH_CONTAINERS event to " + containerId
              + " because container is recovering");
      continue;
    }
    this.dispatcher.getEventHandler().handle(
            new ContainerKillEvent(containerId,
                    ContainerExitStatus.KILLED_BY_RESOURCEMANAGER,
                    "Container Killed by ResourceManager"));
 
  }
  break;
  ```

通过对比日志，印证了上述猜想：
```bash
2019-12-16 18:36:23,551 WARN [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: couldn't find app application_1575307473787_4683518while processing FINISH_CONTAINERS event
2019-12-16 18:36:23,551 WARN [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: couldn't find app application_1575307473787_4683518while processing FINISH_CONTAINERS event
2019-12-16 18:36:23,551 WARN [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: couldn't find app application_1575307473787_4683518while processing FINISH_CONTAINERS event
2019-12-16 18:36:23,551 WARN [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: couldn't find app application_1575307473787_4683518while processing FINISH_CONTAINERS event
 
 
2019-12-16 18:36:23,551 WARN [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: couldn't find container container_1575307473787_14285364_01_006226 while processing FINISH_CONTAINERS event
2019-12-16 18:36:23,551 WARN [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: couldn't find container container_1575307473787_14285364_01_006227 while processing FINISH_CONTAINERS event
2019-12-16 18:36:23,552 WARN [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: couldn't find container container_1575307473787_14285364_01_006228 while processing FINISH_CONTAINERS event
2019-12-16 18:36:23,552 WARN [Thread-5394309] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: couldn't find container container_1575307473787_14285364_01_006229 while processing FINISH_CONTAINERS event
```

接下来，程序进入了while循环等待，并启用一个线程间隔1秒去检测是否所有的container的状态都变成了COMPLETE，如果没有，就继续等待。就是在这里，程序陷入了循环无法退出：

```java

  /*
   * We will wait till all the containers change their state to COMPLETE. We
   * will not remove the container statuses from nm context because these
   * are used while re-registering node manager with resource manager.
   */
  boolean allContainersCompleted = false;
  while (!containers.isEmpty() && !allContainersCompleted) {       <== 代码定位
    allContainersCompleted = true;
    for (Entry<ContainerId, Container> container : containers.entrySet()) {
      if (((ContainerImpl) container.getValue()).getCurrentState()
          != ContainerState.COMPLETE) {  <== 代码定位
        allContainersCompleted = false;
        try {
          Thread.sleep(1000);     <== 代码定位
        } catch (InterruptedException ex) {
          LOG.warn("Interrupted while sleeping on container kill on resync",
            ex);
        }
        break;
      }
    }
  }
  // All containers killed
  if (allContainersCompleted) {
    LOG.info("All containers in DONE state");
  } else {
    LOG.info("Done waiting for containers to be killed. Still alive: " +
      containers.keySet());
  }
}
```

可以看到循环的退出条件是  !containers.isEmpty() && !allContainersCompleted 。很容易看出，!container.isEmpty() 这个已经成立了，那么出问题的就是 !allContainersCompleted 这个条件了。结合上面有找不到Container的日志，调查继续为什么这些Container找不到。以container_1575307473787_14285364_01_006226 为关键字，在日志里查找，发现如下关键信息：

```bash
2019-12-16 18:34:30,470 INFO [AsyncDispatcher event handler] org.apache.spark.network.yarn.YarnShuffleService: Stopping container container_1575307473787_14285364_01_006226
2019-12-16 18:34:34,665 INFO [Container Monitor] org.apache.hadoop.yarn.server.nodemanager.containermanager.monitor.ContainersMonitorImpl: Stopping resource-monitoring for container_1575307473787_14285364_01_006226
2019-12-16 18:35:49,435 INFO [IPC Server handler 2 on 45454] org.apache.hadoop.yarn.server.nodemanager.containermanager.ContainerManagerImpl: Stopping container with container Id: container_1575307473787_14285364_01_006226
2019-12-16 18:35:49,435 INFO [IPC Server handler 2 on 45454] org.apache.hadoop.yarn.server.nodemanager.NMAuditLogger: USER=hdfs IP=10.211.51.90 OPERATION=Stop Container Request        TARGET=ContainerManageImpl     RESULT=SUCCESS  APPID=application_1575307473787_14285364        CONTAINERID=container_1575307473787_14285364_01_006226
```

注意，ResourceResponse要求RESYNC的时间是18:36:23， 而在18:35:49，也就是RESYNC之前，一些Container就已经被STOPPED掉了。这里问题来了，为什么已经STOPPED的Container会出现在 context.getContainers() 中呢？需要继续查找这个Container被stopped之后的逻辑，如下：

```java
private void stopContainerInternal(NMTokenIdentifier nmTokenIdentifier,
    ContainerId containerID) throws YarnException, IOException {
  String containerIDStr = containerID.toString();
  Container container = this.context.getContainers().get(containerID);
  LOG.info("Stopping container with container Id: " + containerIDStr);  <== 代码定位
  authorizeGetAndStopContainerRequest(containerID, container, true,  <== 代码定位
    nmTokenIdentifier);
 
  if (container == null) {
    if (!nodeStatusUpdater.isContainerRecentlyStopped(containerID)) {
      throw RPCUtil.getRemoteException("Container " + containerIDStr
        + " is not handled by this NodeManager");
    }
  } else {
    if (container.isRecovering()){
      throw new NMNotYetReadyException("Container " + containerIDStr + " is recovering, try later");
    }
    context.getNMStateStore().storeContainerKilled(containerID);  <== 代码定位
    dispatcher.getEventHandler().handle(
      new ContainerKillEvent(containerID,
          ContainerExitStatus.KILLED_BY_APPMASTER,
          "Container killed by the ApplicationMaster."));
 
    NMAuditLogger.logSuccess(container.getUser(),    
      AuditConstants.STOP_CONTAINER, "ContainerManageImpl", containerID
        .getApplicationAttemptId().getApplicationId(), containerID);
  }
}
```

看到进入了 authorizeGetAndStopContainerRequest 逻辑。 因为这里stopRequest是传的参数true，所以理论上一定有日志输出，但是实际上没有，所以是 (!nmTokenAppId.equals(containerId.getApplicationAttemptId().getApplicationId()))|| (container != null && !nmTokenAppId.equals(container.getContainerId().getApplicationAttemptId().getApplicationId()))  这个条件没有成立。但是这里没有成立也没有关系，因为没有实质上context更新相关的逻辑，所以问题不是这里。需要继续排查。


```java
@Private
@VisibleForTesting
protected void authorizeGetAndStopContainerRequest(ContainerId containerId,
    Container container, boolean stopRequest, NMTokenIdentifier identifier)
    throws YarnException {
  /*
   * For get/stop container status; we need to verify that 1) User (NMToken)
   * application attempt only has started container. 2) Requested containerId
   * belongs to the same application attempt (NMToken) which was used. (Note:-
   * This will prevent user in knowing another application's containers).
   */
  ApplicationId nmTokenAppId =
      identifier.getApplicationAttemptId().getApplicationId();   <== 代码定位
  if ((!nmTokenAppId.equals(containerId.getApplicationAttemptId().getApplicationId()))  <== 代码定位
      || (container != null && !nmTokenAppId.equals(container
        .getContainerId().getApplicationAttemptId().getApplicationId()))) {
    if (stopRequest) {    <== 代码定位，值为true
      LOG.warn(identifier.getApplicationAttemptId()
          + " attempted to stop non-application container : "
          + container.getContainerId());
      NMAuditLogger.logFailure("UnknownUser", AuditConstants.STOP_CONTAINER,
        "ContainerManagerImpl", "Trying to stop unknown container!",
        nmTokenAppId, container.getContainerId());
    } else {
      LOG.warn(identifier.getApplicationAttemptId()
          + " attempted to get status for non-application container : "
          + container.getContainerId());
    }
  }
}
```


又回到 stopContainerInternal() ， 看到 stopContainers 里面提供了线索，这里在stopContainer之后，往succeededRequests增加了container id，这里是是合理的逻辑，继续跟踪：


```java
@Override
public StopContainersResponse stopContainers(StopContainersRequest requests)
    throws YarnException, IOException {
 
  List<ContainerId> succeededRequests = new ArrayList<ContainerId>();
  Map<ContainerId, SerializedException> failedRequests =
      new HashMap<ContainerId, SerializedException>();
  UserGroupInformation remoteUgi = getRemoteUgi();
  NMTokenIdentifier identifier = selectNMTokenIdentifier(remoteUgi);
  for (ContainerId id : requests.getContainerIds()) {
    try {
      stopContainerInternal(identifier, id);  <== 代码定位
      succeededRequests.add(id);  <== 代码定位
    } catch (YarnException e) {
      failedRequests.put(id, SerializedException.newInstance(e));
    }
  }
  return StopContainersResponse
    .newInstance(succeededRequests, failedRequests);
}
```


继续跟踪，就来到了 ContainerManagementProtocol ，这里是和 ApplicationMaster 进行通信的，很合理。查看日志，如果有问题的话，应该抛出YarnException或者IOException，但是没有找到。所以不是这里的问题，再继续排查。

```java
@Public
@Stable
public interface ContainerManagementProtocol {
 
/**
 * <p>
 * The <code>ApplicationMaster</code> requests a <code>NodeManager</code> to
 * <em>stop</em> a list of {@link Container}s allocated to it using this
 * interface.
 * </p>
 * 
 * <p>
 * The <code>ApplicationMaster</code> sends a {@link StopContainersRequest}
 * which includes the {@link ContainerId}s of the containers to be stopped.
 * </p>
 * 
 * <p>
 * The <code>NodeManager</code> sends a response via
 * {@link StopContainersResponse} which includes a list of {@link ContainerId}
 * s of successfully stopped containers, a containerId-to-exception map for
 * each failed request in which the exception indicates errors from per
 * container. Note: None-container-specific exceptions will still be thrown by
 * the API method itself. <code>ApplicationMaster</code> can use
 * {@link #getContainerStatuses(GetContainerStatusesRequest)} to get updated
 * statuses of the containers.
 * </p>
 * 
 * @param request
 *          request to stop a list of containers
 * @return response which includes a list of containerIds of successfully
 *         stopped containers, a containerId-to-exception map for failed
 *         requests.
 * @throws YarnException
 * @throws IOException
 */
@Public
@Stable
StopContainersResponse stopContainers(StopContainersRequest request)
    throws YarnException, IOException;
```

回头来到 stopContainerInternal , 这里看到还执行了一步 context.getNMStateStore().storeContainerKilled(containerID); 的操作，入参是containerID，那么问题有可能就是这里状态更新有问题：

```java
if (container.isRecovering()){
  throw new NMNotYetReadyException("Container " + containerIDStr + " is recovering, try later");
}
context.getNMStateStore().storeContainerKilled(containerID);   <== 代码定位
dispatcher.getEventHandler().handle(
  new ContainerKillEvent(containerID,
      ContainerExitStatus.KILLED_BY_APPMASTER,
      "Container killed by the ApplicationMaster."));
 
NMAuditLogger.logSuccess(container.getUser(),    
  AuditConstants.STOP_CONTAINER, "ContainerManageImpl", containerID
    .getApplicationAttemptId().getApplicationId(), containerID);
 
 
 
@Override
public void storeContainerKilled(ContainerId containerId)
    throws IOException {
  String key = CONTAINERS_KEY_PREFIX + containerId.toString()
      + CONTAINER_KILLED_KEY_SUFFIX;
  try {
    db.put(bytes(key), EMPTY_VALUE);
  } catch (DBException e) {
    throw new IOException(e);
  }
}
```

# 三、结论

问题原因：handle 中的FINISH_CONTAINER 逻辑陷入了无法退出的循环，因为退出条件allContainerCOMPLETE不成立。

问题根因：context.getContainer 获取的container list 因故没有排除已经STOPPED的container，直接导致这些“幽灵”container无法返回COMPLETE的状态。

解决方案：在context的相关读写步骤，加入DEBUG信息。





