# Amaze 营销活动问题 SOP

> 来源：[优惠券无法使用](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=420579189) + [领券活动页提示无资格](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=352658711) + [优惠券来源排查](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=338366154) + [立减活动问题](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=328509902) + [关闭/恢复用户上课通知](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=780415867) + [通知任务已发送人数为0](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=753732332) + [转化活动问题处理](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=328514645) + [续报活动SOP](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=773373599) + [活动问题自助查询工具](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=958432106) + [上续下落地页问题](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=579126154) + [打包售卖链接无法推荐](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=659274089)
>
> 整理日期：2026-04-30 · 文档记录时状态：全量
> ⚠️ 产品文档不会随功能进展自动更新，实际灰度/上线情况请向产品确认。
>
> 适用角色：电商运营 / 客服 / 技术支持

---

## 一、优惠券相关

### 1.1 优惠券无法使用（下单时没有推荐出优惠券）

**常见原因：** 券的适用范围（班课标签、订单价格范围、联报使用限制类型）与下单班课不匹配。

**排查工具：**

| 工具 | 入口 |
|---|---|
| 优惠券和订单匹配查询（Rush 小工具） | [https://tutor-ec-tool.rush.zhenguanyu.com/#coupon-order-match](https://tutor-ec-tool.rush.zhenguanyu.com/#coupon-order-match) |
| 优惠券和班课匹配查询（Rush 小工具） | [https://tutor-ec-tool.rush.zhenguanyu.com/#coupon-lesson-match](https://tutor-ec-tool.rush.zhenguanyu.com/#coupon-lesson-match) |
| 自助接口（按订单） | `https://tutor.zhenguanyu.com/tutor-uc-tool/api/users/{userId}/coupons/{couponId}/order-no-match-tips?orderIds={orderId}` |
| 自助接口（按班课） | `https://tutor.zhenguanyu.com/tutor-uc-tool/api/users/{userId}/coupons/{couponId}/lesson-no-match-tips?lessonIds={lessonId}` |

接口返回会说明具体不匹配原因（如「班课标签不匹配」「订单价格不满足」等）。若接口返回匹配，则联系研发处理。

> ℹ️ **员工优惠券：** 2025 年员工优惠券金额上调，旧版 6000 元上限限制已不适用。

---

### 1.2 领券活动页提示「无资格」（无法领取优惠券）

**排查步骤：**
1. 确认用户 userId；
2. 查用户领券 HTTP 日志：进入 [Octopus 日志系统](https://octopus.zhenguanyu.com/)，高级模式，查询语句：`service=tutor-lucky-money AND log_type=http AND method=POST AND user_id=【userId】`，找到领取失败日志（状态码 412），定位 `couponJobId`；
3. 在 Amaze 搜索 couponJobId 对应的券规则：[优惠券规则列表](https://amaze.zhenguanyu.com/marketing/#/market/coupon-v2/jobs?page=0)；
4. 在 [用户群组匹配页面](https://amaze.zhenguanyu.com/marketing/#/market/userGroup/userMatcherPage) 检查用户和券规则目标用户是否匹配；
5. 查看领券活动的失败文案配置：[领券活动列表](https://amaze.zhenguanyu.com/marketing/#/market/receiveCouponActivity?page=0) → 进入活动详情页。

> ℹ️ **注意：** 失败提示文案（如"仅限小学用户"）是活动配置的文案，与用户自行设置的年级无关。

---

### 1.3 无法通过 Amaze 查到优惠券来源（老兑换码活动）

**背景：** 老兑换码活动兑换的优惠券在 Amaze 上查不到对应活动信息。

**自助接口（技术支持/产品可用）：**
```
https://tutor.zhenguanyu.com/tutor-uc-tool/api/users/{userId}/coupon-source-info?couponIds={couponId1,couponId2}
```
- 可查到的券 → 接口返回「请到 Amaze 优惠券页面查询」；
- 老兑换码活动的券 → 接口返回「优惠券通过老的兑换码活动兑换（市场年课卡），使用班课范围为：xxx」，可点击范围链接查看。

**优惠券发送入口汇总：**
- Amaze 优惠券活动（发放/领取）
- Amaze 兑换码活动（新）
- 老兑换码活动（不在 Amaze 显示）
- Amaze 员工优惠券

---

## 二、立减活动相关

### 2.1 用户没有享受到立减活动

**常见原因（按顺序排查）：**

| 原因 | 排查方法 |
|---|---|
| 下单时间早于立减活动创建时间 | 查 Amaze 订单操作日志（下单时间）和立减活动操作日志（创建时间） |
| 用户已享受另一个同类但优惠更大的立减 | 在 Amaze 订单详情页查看已享受的立减；在立减活动详情页看互斥规则（同类立减互斥，不同类同享，立减与券同享） |
| 用户不是立减活动目标用户 | 在 Amaze 立减活动详情页找到目标用户群组 ID，在[用户群组匹配页](https://amaze.zhenguanyu.com/#/userGroup/userMatcherPage)查询 |
| 用户不满足下单限制（单报/联报/学季/科目/年级） | 查询历史订单（如调班则以调班链中第一个班课判断） |
| 用户不满足优惠梯度 | 查看立减活动规则（如「第 3 个商品减 50 元」，第 2 个不享受） |

如以上排查后仍无法定位原因，联系研发排查 `tutor-marketing-decrease` 服务日志。

---

## 三、通知任务相关

### 3.1 关闭 / 恢复用户上课通知（短信 / 外呼）

用户可在 App「个人中心 → 设置 → 上课通知」自行操作；若用户无法自行操作，技术支持可通过 XXL-Job 处理。

**XXL-Job 入口：** [CloseUserEpisodeNotifyJob](https://commonxxl.zhenguanyu.com/common-xxl-job-admin/jobinfo?jobGroup=2365)

**参数格式：**
```json
{
  "userId": 168611799,
  "notifyTypes": ["sms", "voice"],
  "close": true
}
```
- `notifyTypes`：`"sms"`（短信）/ `"voice"`（外呼），可单独或组合填写。
- `close`：`true` = 关闭，`false` = 恢复。

**操作步骤：** 点击「执行一次」→ 填写参数 → 保存执行 → 点击「查询日志」确认结果。

> ⚠️ 若页面提示不是 `tutor-student-notify`，说明无权限，联系研发添加。
> ⚠️ `userId` 填用户 ID，不能填手机号；`notifyTypes` 值必须合法（`sms` / `voice`），否则执行失败。

---

### 3.2 通知任务已发送人数为 0

**结论：这是正常产品逻辑。**

**营销类通知发送限制：**

| 通知类型 | 发送上限 |
|---|---|
| Push | 每天 2 个（按 userId × 自然天） |
| 短信 | 每周 1 个（按 userId × 自然周） |
| IM | 不限 |

达到上限则不发送。**另外，每天 19:00–21:00 上课高峰期，所有营销类 Push/短信均不发送。**

**处理方案：** 告知运营错开高峰期时间段发送。

---

## 四、转化活动相关

### 4.1 用户没有拿到转化活动的推荐班课

**排查步骤（技术支持/产品）：**
1. 确认用户手机号 / userId；
2. 确认转化活动 ID：[转化活动列表](https://amaze.zhenguanyu.com/commerce/#/conversionActivity/list)；
3. 确认用户点击的 URL 是否正确（URL 后缀带 `error/2` 会直接提示不满足条件）；
4. 使用 [转化活动查询工具](https://amaze.zhenguanyu.com/commerce/#/conversionActivity/records)，输入用户手机号/ID 查询该用户每次访问转化活动的结果列表，点击「查看」→「查看排除班课」可看到班课被排除的原因。

> ⚠️ 该工具**仅支持 14 天内**的数据；每个范围内都有推荐班课，结果才为成功。

---

## 五、续报活动相关

### 5.1 购买按钮不可点击

**情形一：用户续报班课均已完成购买** → 按钮正常不可点击，无需处理。

**情形二：用户系统时间设置有误** → 引导用户校正设备时间：
- Android 通用：https://support.google.com/android/answer/2841106
- iOS：https://support.apple.com/zh-sg/guide/iphone/iph65f82af3e/ios

### 5.2 活动问题自助查询工具

**Amaze 工具入口：** [https://amaze.zhenguanyu.com/tool#/user-activity-search](https://amaze.zhenguanyu.com/tool#/user-activity-search)

支持模拟用户登录查看其可见的活动页面。若工具显示正常但用户自己打开不正常，大概率是用户登错账号或老师发错链接。

| 场景 | 直接访问链接格式 |
|---|---|
| 上续下活动 | `https://m.yuanfudao.com/developers/{userId}/lessons/unlock/pure?srcLessonId={lessonId}` |
| 续报活动中间页 | `https://m.yuanfudao.com/developers/{userId}/lessons/renewals?actId={actId}` |
| 续报活动详情页 | `https://m.yuanfudao.com/developers/{userId}/lessons/renewals/detail?srcLessonId={lessonId}&actId={actId}&srcTeamId={teamId}` |

**日志查询工具：** [https://amaze.zhenguanyu.com/tool#/user-request](https://amaze.zhenguanyu.com/tool#/user-request)（根据用户 ID 或手机号查询活动接口失败日志，含失败原因）。

---

## 六、上续下落地页问题

### 6.1 打开链接提示「不满足活动条件」

**排查步骤：**
1. 通过 [Amaze 用户搜索](https://amaze-user.zhenguanyu.com/#/users) 按手机号找到 userId；
2. 在 Octopus 查用户请求日志，确认请求的是**专属链接**（含 `srcLessonId=xxx`）还是**通用链接**（无 `srcLessonId`）：
   - 若是专属链接 → 引导用户改用通用链接；
3. 若使用通用链接仍提示不满足，在 Amaze 用户详情页检查该账号下是否有班课（可能用户登的不是上课的账号）；
4. 在 [Amaze 学习计划](https://amaze.zhenguanyu.com/#/jwc/lessonConnections) 搜索用户上学季班课 ID，查看补缴班课是否创建了售卖专用小班。

**链接类型说明：**

| 类型 | 格式 |
|---|---|
| 专属纯解锁落地页 | `https://m.yuanfudao.com/lessons/unlock/pure?srcLessonId=xxx` |
| 专属中间页 | `https://m.yuanfudao.com/lessons/unlock?srcLessonId=xxx` |
| 通用纯解锁落地页 | `https://m.yuanfudao.com/lessons/unlock/pure?keyfrom=unlock` |
| 通用中间页 | `https://m.yuanfudao.com/lessons/unlock?keyfrom=unlock` |

---

## 七、打包售卖链接无法正常推荐班课

### 7.1 排查步骤

1. 确认链接类型（打包售卖链接 vs 预选打包售卖链接）；
2. 确认链接是否携带 `error` 参数；
3. 在 Octopus 查日志：
   ```
   service = tutor-recommend AND ("doRecommend failed, linkId" OR "doRecommend failed for unknown exception")
   ```
   在搜索区追加打包售卖链接 ID。

### 7.2 常见错误日志含义

| 错误日志 | 原因 |
|---|---|
| `packageLessonLink not found` | 链接不存在 |
| `no container found in packageLessonLink` | 打包售卖配置中没有容器 |
| `exist duplicate primaryLessonId` | 存在重复班课 |
| `no valid lesson found in container cause first primary lesson filter` | 容器内主班课全被过滤 |
| `lesson:{} filter by lessonStatus or not external published` | 班课未上架 / 未外部上架 |
| `lesson:{} filter by saleStatus` | 班课未开售 |
| `lesson:{} filter cause lesson sold out` | 班课售罄 |
| `lesson:{} filter cause no recommendable team found` | 找不到可用小班 |
| `doRecommend failed for unknown exception` | 未知原因，联系研发 |

### 7.3 配置修改后手动刷新

打包售卖推荐结果异步加载，每 30 分钟刷新一次。班课配置修改后需手动执行 XXL-Job：

[RefreshPackageSaleLinkRecommendResultJob](https://commonxxl-api.zhenguanyu.com/common-xxl-job-admin/jobinfo?jobGroup=976)

> ⚠️ 若无权限，联系电商研发添加 `tutor-recommend` 服务权限。
