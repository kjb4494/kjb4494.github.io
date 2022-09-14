---
title: JetBrain 계열의 IDE 실행이 안 되는 현상
author: kjb4494
date: 2022-05-04 00:00:00 +0900
categories: [기타, 에러해결]
tags: [jetbrain, error]
---

### JetBrain 계열의 IDE 를 실행해도 로고만 뜬다

최근 윈도우 환경에서 JetBrain 계열 IDE 를 사용할 때 실행조차 되지않고 꺼지거나 로고만 뜨는 경우가 빈번하게 일어났다.

원인을 몰랐을 때, 그냥 캐시 파일들을 지워보거나 IDE 를 재설치 해보거나 재부팅을 해보거나 등등... 여러가지를 시도해봤지만 문제는 전혀 해결되지않았었다. 그러던 중 신기하게도 윈도우 업데이트를 하니까 정상 동작하는걸 경험했다. ...그러나 오래가지 못 해 다시 똑같은 현상이 발생했다 ㅡㅡ;;

본격적으로 원인을 찾아보니 다음과 같은 에러가 로깅되어있었다.

```text
Internal error. Please refer to https://jb.gg/ide/critical-startup-errors

com.intellij.ide.plugins.StartupAbortedException: Cannot start app
    at com.intellij.idea.StartupUtil.lambda$start$15(StartupUtil.java:263)
    at java.base/java.util.concurrent.CompletableFuture.uniExceptionally(CompletableFuture.java:986)
    at java.base/java.util.concurrent.CompletableFuture$UniExceptionally.tryFire(CompletableFuture.java:970)
    at java.base/java.util.concurrent.CompletableFuture.postComplete(CompletableFuture.java:506)
    at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1705)
    at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.exec(CompletableFuture.java:1692)
    at java.base/java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:290)
    at java.base/java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(ForkJoinPool.java:1020)
    at java.base/java.util.concurrent.ForkJoinPool.scan(ForkJoinPool.java:1656)
    at java.base/java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1594)
    at java.base/java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:183)
Caused by: java.net.BindException: Address already in use: bind
    at java.base/sun.nio.ch.Net.bind0(Native Method)
    at java.base/sun.nio.ch.Net.bind(Net.java:459)
    at java.base/sun.nio.ch.Net.bind(Net.java:448)
    at java.base/sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:227)
    at io.netty.channel.socket.nio.NioServerSocketChannel.doBind(NioServerSocketChannel.java:134)
    at io.netty.channel.AbstractChannel$AbstractUnsafe.bind(AbstractChannel.java:562)
    at io.netty.channel.DefaultChannelPipeline$HeadContext.bind(DefaultChannelPipeline.java:1334)
    at io.netty.channel.AbstractChannelHandlerContext.invokeBind(AbstractChannelHandlerContext.java:506)
    at io.netty.channel.AbstractChannelHandlerContext.bind(AbstractChannelHandlerContext.java:491)
    at io.netty.channel.DefaultChannelPipeline.bind(DefaultChannelPipeline.java:973)
    at io.netty.channel.AbstractChannel.bind(AbstractChannel.java:260)
    at io.netty.bootstrap.AbstractBootstrap$2.run(AbstractBootstrap.java:356)
    at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:164)
    at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:469)
    at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:503)
    at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:986)
    at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
    at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
    at java.base/java.lang.Thread.run(Thread.java:829)
```

`Caused by: java.net.BindException: Address already in use: bind` 원인은 JetBrain 사의 IDE 가 사용하는 포트를 hyper-v 가 점유하고 있어서 실행이 안 되는거였다.

### 해결방법

1. cmd 를 관리자모드로 실행
2. hyper-v 비활성화
   ```shell
   dism.exe /Online /Disable-Feature:Microsoft-Hyper-V
   ```
3. hyper-v 에 다른 포트를 할당
   ```shell
   netsh int ipv4 add excludedportrange protocol=tcp startport=50151 numberofports=1
   ```
4. hyper-v 재활성화
   ```shell
   dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All
   ```

3번에서 만약 `The process cannot access the file because it is being used by another process.` 메세지를 반환하면 `startport=50151` 부분을 다른 포트로 변경해줘야한다. `Ok.` 가 떴다면 정상으로 변경된 것이다.
