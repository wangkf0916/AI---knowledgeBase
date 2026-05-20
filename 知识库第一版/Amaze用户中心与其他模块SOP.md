# Amaze 用户中心与其他模块操作 SOP

> 来源：[多设备互踢](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=811952704) + [更换用户手机号](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=519313022) + [修改昵称提示敏感词](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=449721324) + [用户Profile常见问题](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=153308144) + [查询用户与群组匹配关系](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=327295420) + [更换主讲老师 LDAP](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=328510674) + [用户购课被限制](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=496617851) + [发票开票失败](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=569835173) + [订单物流页不展示订单](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=531031978) + [学习计划联动调班](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=540913733) + [教育部显示时间不一致](https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=542586298)
>
> 整理日期：2026-04-30 · 文档记录时状态：全量
> ⚠️ 产品文档不会随功能进展自动更新，实际灰度/上线情况请向产品确认。
>
> 适用角色：电商运营 / 客服 / 技术支持

---

## 一、用户中心操作

### 1.1 多设备互踢问题处理

**问题现象：** 用户进入已登录的 App 后出现弹窗提示自动退出登录。

**两种互踢逻辑：**

| 互踢类型 | 触发条件 | 提示文案 |
|---|---|---|
| **教室内互踢** | 同一教室同时有多个设备进入，先进入的设备被踢 | "同一账号已在其他设备登录" |
| **教室外互踢** | 同一类型 App 登录设备数超过 **4 台**，再次进入第一台时触发 | "当前账号登录设备超过4台，将自动退出登录" |

> ⚠️ 猿辅导、小猿优课、小猿素养是**不同 App，分别计数**。

**排查方法：**

教室外互踢日志查询：
```
project = tutor-student-app AND 用户设备数量超出限制 AND "userId=361482186"
```
日志中包含用户已登录设备信息和被踢出设备信息。

查询用户当前已登录设备列表：
```
https://tutor.zhenguanyu.com/tutor-atm-user/api/user-login-devices/{userId或手机号}
```

---

### 1.2 更换用户手机号

> ⚠️ 老 CRM 站点更换手机号页面已下线，请使用 **Amaze 学生详情页**操作。

**新入口：** Amaze → 账号 → 学生 → 用户详情页

**修改方式：**
1. 用户可在猿辅导 App 自行修改手机号；
2. 客服可在 Amaze 后台为用户修改手机号。

**常见问题：**
- **页面打不开** → 确认是否访问的是老页面，引导使用 Amaze 学生详情页；若新页面无法访问，联系产品配置权限。
- **非客服运营** → 无权限使用该功能，须在 Just 提工单联系客服处理。

适用客服：辅导客服、斑马客服。

#### 查询用户手机号更变记录

1. 进入 **Amaze 用户详情页 → 帐号信息 Tab**，查看「手机号变更记录」表格（LOG1）；
2. 进入 **Amaze 用户详情页 → 操作日志 Tab**，查看客服后台操作的手机号修改日志（LOG2）；
3. 出现在 LOG1 但**不在** LOG2 的记录 = 用户在 App 上自行修改的。

如需进一步定位，查询 `fenbi-auth` 日志：
```
service=fenbi-auth AND "com.fenbi.auth.server.service.impl.AccountServiceImpl" AND Phone AND 【userId 或新/老手机号】
```
日志中会显示旧手机号 → 新手机号的变更记录；用日志中的 `traceId` 可进一步追踪完整链路：
```
trace_id=【traceId】
```

---

### 1.3 修改昵称提示敏感词

> 敏感词使用阿里云和 fenbi-list 控制，可能出现误判，需敏感词服务负责人手动处理。

**修改昵称的两种方式：**
1. 用户在猿辅导 App / 网课自行修改；
2. 客服/运营在 Amaze 后台修改。

**排查步骤：**

查询 `fenbi-spam` 日志，确认命中类型：
```
service = ape-fenbi-spam AND 【用户要修改的昵称内容】
```

| 日志特征 | 命中类型 | 处理方式 |
|---|---|---|
| 包含 `com.fenbi.spam.server.proxy.AliGreenProxy` | 命中阿里云策略 | 在钉钉群「【外】猿辅导内容安全交流」反馈（如未进群联系相关同学）|
| 包含 `text blocked by local rules: content = xxx` | 命中本地规则 | 联系 fenbi-spam 负责人处理 |
| 其他情况 | 未知 | 在企业微信群「【敏感词报障】」反馈 |

#### 查询用户昵称修改记录

1. 查询 `fenbi-profile` 昵称变更日志：
```
service=ape-fenbi-profile AND NOT log_type=rpc AND "NICKNAME_CHANGE" AND 【userId】
```
2. 若是客服修改，在 **Amaze 用户详情页 → 操作日志**中查看；
3. 若是用户自行修改，用步骤 1 获取的 `traceId` 查询 HTTP 日志：
```
service=ape-fenbi-profile AND NOT log_type=rpc AND addOrUpdateUserInfo AND 【userId】 AND trace_id=【traceId】
```
4. 若无 HTTP 日志 → 为其他业务线（如 QWERTY 等）修改的昵称。

---

### 1.3-补充 修改用户头像无效果

**背景：** 用户在 App 修改头像时，会调用 `ape-fenbi-spam` 审核图片，只有审核通过的图片才能设置为头像，否则重置为默认头像。

**排查步骤：**

**第一步：** 查询 `fenbi-profile` HTTP 日志，确认修改请求：
```
service=ape-fenbi-profile AND NOT log_type=rpc AND addOrUpdateUserInfo AND 【userId】
```

**第二步：** 查询头像审核结果，获取被拒绝的图片 URL：
```
service=ape-fenbi-profile AND "SPAM_CHECKED" AND 【userId】
```
日志示例：`@@@SPAM_CHECKED@@@ userId=xxx, url=https://gallery.fbcontent.cn/api/ape/images/xxx.jpg`

**第三步：** 查询 spam 审核详细结果：
```
service=ape-fenbi-spam AND "com.fenbi.spam.server.proxy.AliGreenProxy" AND 【第二步中的图片文件名】
```
日志中查看 `suggestion` 字段：
- `block` → 明确拒绝
- `review` 且 `scene=porn` → 也拒绝

---

### 1.4 查询用户与用户群组的匹配关系

**使用场景：** 排查售卖策略、立减服务、优惠券、资源位等是否对某用户生效时，目标用户为用户群组的情况。

**操作工具：** [Amaze 用户群组匹配查询页面](https://amaze.zhenguanyu.com/#/userGroup/userMatcherPage)

**操作逻辑：**
1. 在 Amaze 对应功能详情页（立减活动/售卖策略等）找到目标用户群组 ID；
2. 在匹配查询页面输入用户和群组 ID；
3. 若不匹配，工具会提示到子元素（子群组/用户标签）层级，可逐层向下排查。

**用户标签不匹配的常见处理：**

| 标签类型 | 处理方 |
|---|---|
| Leads 中心标签 | 联系用户增长同学 |
| 上传 CSV 文件的标签 | 确认用户是否在 CSV 名单中；若在但仍不匹配，联系电商同学排查 |
| 购课标签 | 检查用户订单是否满足标签的班课条件；满足但仍不匹配，联系电商同学排查 |
| 其他标签 | 联系电商同学处理 |

**手动刷新用户 OrderTag（研发操作）：** 确认用户满足购课标签条件后，在 XXL-Job 执行 `ManualUpdateOrderTagByUserIdsJob`，入参 `userIds` 填用户 ID（支持多个）：[任务入口](https://commonxxl.zhenguanyu.com/common-xxl-job-admin/jobinfo?jobGroup=906)

---

### 1.5 更换主讲老师 LDAP

**换绑操作：**
1. 访问 [https://tutor-debug.zhenguanyu.com/#/tool-bind-teacher-ldap](https://tutor-debug.zhenguanyu.com/#/tool-bind-teacher-ldap)；
2. 输入 `teacherId` 和新 LDAP，点击「换绑」；
3. 在 Amaze 老师详情页确认操作结果。

**解绑 LDAP：**
1. 访问 [https://tutor-debug.zhenguanyu.com/#/tool-clear-teacher-ldap](https://tutor-debug.zhenguanyu.com/#/tool-clear-teacher-ldap)；
2. 输入 `teacherId`，点击「解绑」；
3. 在 Amaze 老师详情页确认操作结果。

---

### 1.6 用户购课被限制（无购买资格）

**自助排查接口：**
```
https://tutor.zhenguanyu.com/tutor-uc-tool/api/sale-strategy/user/{userId}/purchase?lessonIds={lessonId1,lessonId2}
```
接口返回会说明用户被哪个售卖策略/用户群组限制，以及不匹配的原因。

**带特殊渠道的售卖策略查询（加参数）：**
```
?lessonIds=xxx&productId=301&hostProductId=301&keyfrom=123
```

**补充查询工具：**
| 工具 | 入口 |
|---|---|
| 售卖策略列表 | [https://amaze.zhenguanyu.com/commerce/#/saleStrategyDispatch/lessonSaleStrategyPage](https://amaze.zhenguanyu.com/commerce/#/saleStrategyDispatch/lessonSaleStrategyPage) |
| 用户群组查询 | [https://amaze.zhenguanyu.com/marketing/#/market/userGroup/userGroupPage](https://amaze.zhenguanyu.com/marketing/#/market/userGroup/userGroupPage) |
| 用户是否满足群组 | [https://amaze.zhenguanyu.com/marketing/#/market/userGroup/userMatcherPage](https://amaze.zhenguanyu.com/marketing/#/market/userGroup/userMatcherPage) |

> ⚠️ 若当时触发了限购，但现在自助工具显示可以买，可在 [Octopus](https://octopus.zhenguanyu.com/) 查询当时的 `tutor-strategy` 服务请求日志确认。若查不到，可能是用户登录账号与给的手机号不同。

---

## 二、订单与履约

### 2.1 App 订单物流页不展示某订单

**问题原因：** App 订单物流页会根据**班课标签的展示平台**属性进行过滤，未配置对应 App 展示的班课标签不会展示。

**排查步骤：**
1. 在 Amaze 班课详情页查看该班课的**班课标签名称**；
2. 在 [班课标签列表页](https://amaze.zhenguanyu.com/#/jwc/lessonPropsManage/marks) 按名称搜索该标签，查看「展示平台」配置；
3. 若展示平台没有对应的 App → 基本确认是此原因，咨询班课对应教务老师确认标签展示平台配置是否正确，或找产品确认；
4. 若展示平台有对应 App 仍不展示 → 提 Darwin Case 给电商同学排查。

---

### 2.2 学习计划联动调班问题

**产品功能：** 用户调班时，若上下学季班课存在学习计划绑定关系，会自动触发联动调班。

**学习计划绑定关系查询：** [https://amaze.zhenguanyu.com/#/jwc/lessonConnections](https://amaze.zhenguanyu.com/#/jwc/lessonConnections)

**排查步骤（通过 Octopus 日志）：**
1. 查询 `tutor-student-lesson` 服务日志，搜索 `lesson-connection AND {lessonId} AND {userId}`，查看 `lesson-connection` 日志；
2. 获取 traceId，确认是否 `true`（会触发联动调班）；
3. 查询 `tutor-student-order` 服务日志，搜索 `transferLesson AND {userId}`，查看 `transferWithLessonConnection=true` 的日志；
4. 用 traceId 查联动调班结果和失败原因。

**报障模板：**
```
描述：用户id=xxx，原班(lessonId=A)调到(lessonId=B)，下学季班课(lessonId=C)没有联动调班
订单链接：xxx
截图：xxx
```

---

## 三、客服与其他

### 3.1 发票开票失败错误原因查询

> ⚠️ 开票失败的发票**目前仅支持客服线下帮助重开**，无法在 JUST 1.0 平台或用户端重开。

**步骤一：在 JUST 1.0 查询失败发票**
- 入口：[https://kefu.zhenguanyu.com/#/tools/receipts-core](https://kefu.zhenguanyu.com/#/tools/receipts-core)
- 若无权限，联系产品老师开通 JUST 1.0 权限

**步骤二：若 JUST 1.0 无记录，查 Octopus 日志**
```
service = tutor-ec-misc AND "false" AND "consumer" AND "userId":【用户userId】
```
注意：日志最多保留 **1 个月**内的数据。

**常见错误原因说明：**

| 错误原因 | 解释 | 解决方案 |
|---|---|---|
| 获取下一张发票代码号码失败，请查看库存是否为空 | 三方电子发票用完了 | 客服线下联系财务手动开票 |
| 金税盘被拔出 / 设备中没有找到金税盘 | 三方服务掉线 | 等待三方恢复 |
| 请稍后进行手工重推 | 中台或三方问题，暂不支持重推 | 联系研发 |
| 设备通讯异常 / 设备不在线 | 三方断网 | 等待恢复 |
| 金税盘处于汇总期 / 正在抄报税 | 三方对账中 | 等待抄报完成后重试 |
| 购方税号不能全为0 | 税号传参错误 | 检查用户填写的税号 |
| 本次操作需登录电子发票服务平台 | 中台掉线 | 联系研发重新登录 |

---

### 3.2 辅导 App 班课课程时间与教育部显示时间不一致

**结论：这是正常的产品配置策略，无需修复。**

**原因：** 为降低教育部平台配置运营成本，相同年级+科目的多个时间段班课在教育部平台只配置了**一个**班课（不一一对应）。用户在辅导 App 内可查看实际购买的班课，**上课时间以 App 内展示为准**。

参考 Darwin Issue：[#354767](https://darwin.zhenguanyu.com/#/issues/354767?biz=tutor)
