# 进教室 Error Code 定义

> 来源：[Confluence — 进教室error code定义](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=95606329)（版本 v25，空间：猿辅导-课程体验）
>
> 相关文档：[进教室步骤及关键路径](https://shimo.zhenguanyu.com/docs/T3RK3jJJGD3JqHx3)
>
> 整理日期：2026-04-30 · 文档记录时状态：全量
>
> 适用角色：客户端开发 / 技术排障

---

## 一、设计原则

1. 以 `C_` 开头，表示客户端自定义 ErrorCode，与服务端 ErrorCode 区分；
2. 将错误分成几大类，再针对不同场景细分 SubCode，便于快速定位原因；
3. 若本身的 ErrorMessage 中已包含「错误码」字段，则不再添加；
4. 同一时期引发多个错误码时，按以下优先级顺序显示：
   > **最高优先级 > 引擎 Error > roomError > enterRoom.errorMessage**
   >
   > ⚠️ 引擎 Error 不属于本次定义的客户端 Error，其自带错误码，未列在表中。
5. 若同时触发多个同一优先级的 Error，只显示第一个，避免页面文字闪动；

---

## 二、查询方式

> 目前每个 errorCode 只对应一个场景。

1. 先从 `error-center.service.ts` 中找到该 errorCode 对应的变量名；
2. 查找该变量名使用的场景；
3. 对应场景的上下文中均有详细的 `clog`，errorCode 只作为提示，具体 reason 参考 clog。

---

## 三、Error Code 对照表

### 001 — 网络异常

| SubCode | 完整错误码 | 提示文案 | 触发场景 | 代码位置 | 优先级 |
|---|---|---|---|---|---|
| 01 | `C_001-01` | 网络异常，请确认网络连接失败后重试(错误码:C_001-01) | 回放 & 未连网 | — | 最高优先级 |
| 02 | `C_001-02` | 回放获取失败(错误码:C_001-02) | 回放 & chunk 下载失败 | `packet-barrier.service.ts` → `afterConsumingPacket` → `downloadChunk` | roomError |
| 03 | `C_001-03` | 回放获取失败(错误码:C_001-03) | 回放 & prepare 失败（`snapshotInfo.info.index === -1`） | `replay-player.service.ts` → `prepare` → `seek` | roomError |
| 04 | `C_001-04` | 回放获取失败(错误码:C_001-04) | 回放 & prepare 失败（`snapshotInfo.needAdminExclusiveInfo && snapshotInfo.adminExclusiveInfo.index === -1`） | `replay-player.service.ts` → `prepare` → `seek` | roomError |
| 05 | `C_001-05` | 回放获取失败(错误码:C_001-05) | 回放 & `seekImpl` 失败 | `replay-player.service.ts` → `seekImpl` | roomError |

---

### 002 — API 失败

| SubCode | 完整错误码 | 提示文案 | 触发场景 | 代码位置 | 优先级 |
|---|---|---|---|---|---|
| 01 | ~~`C_002-01`~~ | ~~教室连接失败，请检查网络(错误码:C_002-01)~~ | ~~直播 & 辅导老师获取 team 接口失败~~（已废弃） | — | ~~roomError~~ |
| 02 | `C_002-02` | 回放获取失败(错误码:C_002-02) | 回放 & 获取回放数据接口失败（`tutor-replay/win/episodes/${episodeId}/versional-replay/compatible-info?replayClientVersion=${ReplayClientVersion}`） | `replay.service.ts` → `getReplayCompatibleInfo` | enterRoom.errorMessage |
| 03 | `C_002-03` | 回放获取失败(错误码:C_002-03) | 回放 & 获取回放数据接口失败（`tutor-replay/win/episodes/${episodeId}/versional-replay?dataVersion=${dataVersion}`） | `replay.service.ts` → `getReplayInfo` | enterRoom.errorMessage |
| 04 | `C_002-04` | 回放获取失败(错误码:C_002-04) | 回放 & 获取回放数据接口失败（`tutor-replay-gateway/win/replay/episodes/${episodeId}/config`） | `replay.service.ts` → `getReplayConfigInfo` | enterRoom.errorMessage |
| 05 | `C_002-05` | 回放转直播信息获取失败(错误码:C_002-05) | 获取回放转直播信息失败（`/tutor-room-resource/{client}/rooms/{roomId}/replay-to-live-info`） | `classroom.service.ts` → `getReplayToLiveInfo` | enterRoom.errorMessage |

---

### 003 — 参数异常

| SubCode | 完整错误码 | 提示文案 | 触发场景 | 代码位置 | 优先级 |
|---|---|---|---|---|---|
| 01 | `C_003-01` | 获取服务器列表失败(错误码:C_003-01) | 直播 & 获取引擎参数失败 | — | enterRoom.errorMessage |
| 02 | `C_003-02` | 教室连接失败，请检查网络(错误码:C_003-02) | 回放 & 没有 RoomInfo | — | enterRoom.errorMessage |
| 03 | `C_003-03` | 当前版本过低，请升级后重试(错误码:C_003-03) | 回放 & 缺少对应的 featureSet | — | enterRoom.errorMessage |
| 04 | `C_003-04` | 教室连接失败，请检查网络(错误码:C_003-04) | 回放 & 连接回放引擎失败（通过 `setStreamInfo` 返回值判断） | — | enterRoom.errorMessage |

---

### 009 — 其他

| SubCode | 完整错误码 | 提示文案 | 触发场景 | 代码位置 | 优先级 |
|---|---|---|---|---|---|
| 01 | `C_009-01` | 教室连接失败，请检查网络(错误码:C_009-01) | 直播 & 连接直播引擎失败 | — | enterRoom.errorMessage |
| 02 | `C_009-02` | 因客户端原因暂不支持本课程播放，请暂时移步猿辅导移动端 APP 中查看，我们将尽快解决该问题(错误码:C_009-02) | 回放 & prepare 失败 & reason = 太老的回放课 | `replay-room.component.ts` → `ngOnInit` → `this.replayPlayer.prepare(xx).catch` | enterRoom.errorMessage |
| 03 | `C_009-03` | 回放获取失败(错误码:C_009-03) | 回放 & prepare 失败 & reason = 其他 | — | enterRoom.errorMessage |
| 04 | `C_009-04` | 回放获取失败(错误码:C_009-04) | 引擎连接 error | `enter-room.service.ts` → `tackLiveEngineDisconnectOrError` | enterRoom.errorMessage |
| 05 | `C_009-05` | 回放获取失败(错误码:C_009-05) | 回放 & prepare 失败 & reason = `WRONG_NPT` | `replay-room.component.ts` → `ngOnInit` → `this.replayPlayer.prepare(xx).catch` | enterRoom.errorMessage |
| 06 | `C_009-06` | 回放获取失败(错误码:C_009-06) | 回放 & prepare 失败 & reason = `PROTO_NOT_FOUND` | — | enterRoom.errorMessage |
| 07 | `C_009-07` | 回放获取失败(错误码:C_009-07) | 回放 & prepare 失败 & 缺少 reason（理论上不会出现，作为初始化用） | — | enterRoom.errorMessage |

---

## 四、优先级说明

| 优先级层级 | 说明 |
|---|---|
| **最高优先级** | 无网络连接等基础环境异常（如 C_001-01） |
| **引擎 Error** | 引擎自带错误码，不在本表范围内 |
| **roomError** | 房间级别错误（如 C_001-02 ~ C_001-05） |
| **enterRoom.errorMessage** | 进教室流程中的接口/参数/连接失败（002/003/009 系列） |
