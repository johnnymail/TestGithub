```mermaid
flowchart LR
  %% ===== Consensus / Networking =====
  subgraph C[Consensus & Networking: HotStuff BFT / PoS / VRF / BLS]
    VRF[VRF random beacon\nselect proposer / rotation]
    BLS[BLS aggregated signature\ncompress QC]
    HS[HotStuff-like 2-chain BFT\n~1s block, instant finality]
    VRF --> HS --> BLS
  end

  %% ===== Data Layers =====
  subgraph D[Data availability & state persistence]
    L[Ledger (ordered blocks)]
    TDB[Triedb (KV) + Hexary MPT\nstate_root commitment]
    AUX[Aux index rebuild\n(txhash->position, topic->position)]
    L -->|"replay exec\nbatch write outputs"| TDB
    TDB -->|"export / refresh"| AUX
  end

  %% ===== Accounts / Objects =====
  subgraph S[Accounts & objects]
    EOA[EOA external account\nsecp256k1]
    ACT[Actor smart agent\nPython code + private KV state]
    PVM[Python VM (PVM)\ndeterministic subset]
    EOA -->|send tx| ACT
    ACT -->|run code| PVM
  end

  %% ===== Messaging / Mailbox =====
  subgraph M[Async messaging & mailbox]
    MSG[Message (async)\nsmall data / transfer]
    MB[Mailbox\nordered consume, exactly-once]
    ACT -->|send_message| MSG --> MB --> ACT
  end

  %% ===== Timers / Scheduler =====
  subgraph T[Scheduler: native timers]
    TQ1[Block Ring Buffer\nnear-term O(1) dequeue]
    TQ2[Epoch Queue\nmigrate by epoch]
    TQ3[Overflow Sorted Set\nfar-future timers]
    GBA[Gas Bidding Agent\nbasefee/congestion/balance aware]
    TW[TimeWheel + WatchMap\n(height/timestamp/state triggers)]
    TQ3 --> TQ2 --> TQ1
    TW -->|"EOB candidate collection"| GBA -->|"enqueue by tip + aging"| MB
  end

  %% ===== Off-chain Compute =====
  subgraph O[Verifiable off-chain compute]
    DISP[OffchainTaskDispatcher\ntask def + result schema]
    REG[RunnerRegistry\nstake / heartbeat / health]
    RUN[Runner executor\nlocal VRF self-selection]
    SUB[RunnerSubmission\nsubmit result / threshold agg]
    DEFER[Deferred Callback\nbuild delayed tx -> timers]
    ACT -->|"submit_task\n(with result_schema)"| DISP
    DISP --> REG
    REG --> RUN -->|"execute or skip_task"| SUB --> DEFER --> MB --> ACT
  end

  %% ===== Entitlements / Security =====
  subgraph E[Entitlements]
    REQ[Actor requires[]\n(e.g., net.http, sec.tee)]
    PROV[Runner provides[]\n(declared capabilities)]
    ENF[Runtime enforcement & audit\ngrant/upgrade/revoke traceable]
    REQ --> ENF
    PROV --> ENF
  end

  %% ===== Fees / Economics =====
  subgraph F[Economics & fees]
    CYC[Cycles (compute units)]
    CEL[Cells (byte/data units)]
    BF[EIP-1559 style basefee\nper-block feedback, basefee burn]
    TIP[Tips -> proposer/validators]
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
