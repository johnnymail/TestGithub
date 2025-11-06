```mermaid
flowchart LR
  %% ===== Consensus / Networking =====
  subgraph C[共识与网络 (HotStuff BFT · PoS · VRF · BLS)]
    VRF[VRF 随机信标\n决定提议者/轮换]
    BLS[BLS 聚合签名\n压缩 QC]
    HS[HotStuff 风格两链BFT\n~1s出块, 即时最终性]
    VRF --> HS --> BLS
  end

  %% ===== Data Layers =====
  subgraph D[数据可用性与状态持久化]
    L[Ledger（区块顺序日志）]
    TDB[Triedb（KV）+ Hexary MPT\nstate_root 承诺]
    AUX[Aux 重建索引\n(txhash→位置, 主题→位置)]
    L -->|"执行回放\n产出批量写入"| TDB
    TDB -->|"导出/刷新"| AUX
  end

  %% ===== Accounts / Objects =====
  subgraph S[账户与对象模型]
    EOA[EOA 外部账户\nsecp256k1]
    ACT[Actor 智能体\nPython 程序 + 私有KV状态]
    PVM[Python VM (PVM)\n确定性子集]
    EOA -->|发起交易| ACT
    ACT -->|代码运行| PVM
  end

  %% ===== Messaging / Mailbox =====
  subgraph M[异步消息与邮箱]
    MSG[Message 异步消息\n可携带少量数据/转账]
    MB[Mailbox 邮箱\n顺序消费, exactly-once]
    ACT -->|send_message| MSG --> MB --> ACT
  end

  %% ===== Timers / Scheduler =====
  subgraph T[SCHEDULER 原生定时器与调度]
    TQ1[Block Ring Buffer\n近端O(1)出队]
    TQ2[Epoch Queue\n按纪元批量迁移]
    TQ3[Overflow Sorted Set\n远期定时]
    GBA[Gas Bidding Agent\n上下文竞价(基费/拥塞/余额)]
    TW[TimeWheel + WatchMap\n(高度/时间戳/状态触发)]
    TQ3 --> TQ2 --> TQ1
    TW -->|"EOB 收敛候选"| GBA -->|"按tip+aging入队"| MB
  end

  %% ===== Off-chain Compute =====
  subgraph O[可验证 Off-chain 计算]
    DISP[OffchainTaskDispatcher\n任务定义+结果Schema]
    REG[RunnerRegistry\n质押/心跳/健康度]
    RUN[Runner 执行者\n本地VRF自证选中]
    SUB[RunnerSubmission\n提交结果/阈值聚合]
    DEFER[Deferred Callback\n构造延迟事务→交由定时器]
    ACT -->|"submit_task\n(含result_schema)"| DISP
    DISP --> REG
    REG --> RUN -->|"执行或 skip_task"| SUB --> DEFER --> MB --> ACT
  end

  %% ===== Entitlements / Security =====
  subgraph E[Entitlements 权限框架]
    REQ[Actor requires[]\n(如 net.http, sec.tee)]
    PROV[Runner provides[]\n(声明可提供能力)]
    ENF[运行时强制与审计\n授予/升级/吊销可追溯]
    REQ --> ENF
    PROV --> ENF
  end

  %% ===== Fees / Economics =====
  subgraph F[经济与费用机制]
    CYC[Cycles 计算度量]
    CEL[Cells 字节/数据度量]
    BF[EIP-1559 双基费\n按块反馈调节, 基费焚烧]
    TIP[小费→提议者/验证者]
    CYC --> BF --> TIP
    CEL --> BF
  end

  %% ===== Cross-links =====
  HS --> L
  L -. header.state_root .-> TDB
  PVM --> TDB
  MB --> TDB
  O --> TDB
  E --> O
  F --> HS
  F --> PVM
```
