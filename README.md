```flowchart
flowchart TD
    A[用户触发进教室 OnEnterClass] --> B{录播等特殊课型?}
    B -->|是| C[关闭所有多开子进程<br/>清模块信息 / ClearRoomData<br/>主进程 PreEnterClass]
    B -->|否| D{正在 bEntering?}
    D -->|是| E[直接 return]
    D -->|否| F[_CheckEnterRoomData 拉教室信息]

    F --> G{新教室 enableMultiClassroom<br/>且角色为助教?}
    G -->|否| H[SynchLeaveRoom]
    H --> I{新教室未开多开?}
    I -->|是| J[invokeCloseAllMultiClient]
    I -->|否| K[跳过关子进程]
    J --> L{进房接口成功?}
    K --> L
    L -->|失败| M[提示错误 / 回调失败]
    L -->|成功| N[主进程进教室流程<br/>Remove旧cid / 写新cid<br/>ClearRoomData / PreEnterClass]

    G -->|是| O{已在某教室<br/>且目标 cid 不同?}
    O -->|否| H
    O -->|是| P{当前主进程教室数据<br/>EnableMultiClassroom?}
    P -->|否| Q[invokeCloseAllMultiClient]
    P -->|是| R[不关子进程]
    Q --> S[ModuleInfo Append 新教室信息]
    R --> S
    S --> T[invokeLaunchMultiClientClass<br/>起新 exe 带 --mtopen 等参数]
    T --> U[本函数 return<br/>主进程不 SynchLeaveRoom 换房]

    subgraph sub1 [子进程侧]
        T --> V[新进程 MultiClientToEnterClass]
        V --> W[读 enter_param Base64]
        W --> X[LoadLoginUserData]
        X --> Y[invokeEnterClass_NoLog 进指定教室]
    end

    subgraph sub2 [主进程对已起子进程]
        T --> Z[_roomsPInfo 记录 pid]
        Z --> AA{该 cid 进程已存在?}
        AA -->|是| AB[QtMultiClientHost<br/>raise_window_noop 置顶]
        AA -->|否| AC[ExePluginTool 启动新进程]
    end
```
