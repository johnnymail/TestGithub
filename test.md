```mermaid

flowchart LR
  subgraph Dev[开发者]
    D1[编写 Actor 代码<br/>声明 requires[] 权限]
    D2[部署 Actor/初始化状态<br/>publish_actor]
    D3[发起交易/调用方法<br/>send_tx()]
    D4[注册定时器/发送异步消息<br/>set_timer()/send_message()]
    D5[发起 Off-chain 任务<br/>submit_task(schema)]
    D6[处理回调/消息<br/>handle(message/callback)]
  end

  subgraph Runtime[链上运行时]
    R0[Tx Pool/内存池]
    R1[PVM 执行器<br/>确定性子集]
    R2[Mailbox 邮箱<br/>顺序消费·幂等]
    R3[Timer 原生调度<br/>Tiered Queue/TimeWheel]
    R4[Offchain Dispatcher<br/>任务路由/结果校验]
    R5[Entitlements 权限<br/>授予/审计/吊销]
    R6[Triedb 状态树<br/>state_root 承诺]
    R7[收据/日志/索引]
  end

  subgraph OffC[Off-chain 执行者]
    O1[Runner Registry<br/>质押/心跳/健康度]
    O2[Runner 集群<br/>执行→提交结果]
  end

  D1 --> D2 --> D3 --> R0 --> R1
  D4 --> R1
  D5 --> R4
  R1 --> R2 --> D6
  R1 --> R3 --> R2
  R4 --> O1 --> O2 --> R4 --> R2 --> D6
  R1 --> R5
  R1 --> R6 --> R7


```
