# Home Assistant 接入美的 / 华凌空调 完整教程（云端版）

> 适用于 **美的、华凌（WAHIN）** 等美的系空调。华凌是美的旗下品牌，账号体系共用美的美居。
>
> 本教程使用云端集成 [sususweet/midea_auto_cloud](https://github.com/sususweet/midea_auto_cloud)，**无需局域网 token、无需拆机抓包**，登录美的美居账号即可把账号下所有美的/华凌设备接入 Home Assistant。

## 相关仓库（同一系列教程）

- [cbywyz/ha-hualing-fan-broadlink](https://github.com/cbywyz/ha-hualing-fan-broadlink) —— 华凌**风扇**（WH-FGA2401）红外接入教程：Broadlink RM3 学码 + 全套遥控器编码库。风扇没有 WiFi，走不了本教程的云端方案，遥控器这些键的码都收录了。
- [cbywyz/gree-yapqf-broadlink-smartir](https://github.com/cbywyz/gree-yapqf-broadlink-smartir) —— 格力空调（YAPQF）Broadlink + SmartIR 接入教程：本地红外控制路线，含码表坑排查与学码补键。
- [cbywyz/phicomm-aircat-m1](https://github.com/cbywyz/phicomm-aircat-m1) —— 斐讯悟空 M1 空气检测仪本地复活（自建集成 + 温湿度/PM2.5/甲醛），空调面板的温湿度数据源好搭档。

## 为什么华凌要用云端方案？

| 方案 | 集成 | 美的 | 华凌 | 说明 |
|------|------|------|------|------|
| 本地局域网 | [wuwentao/midea_ac_lan](https://github.com/wuwentao/midea_ac_lan) | ✅ | ❌ | 华凌新固件走 **V3 协议**，token 无法从美居云端 API 获取，本地接入死结（见 [issue #275](https://github.com/wuwentao/midea_ac_lan/issues/275)） |
| **云端（本教程）** | [sususweet/midea_auto_cloud](https://github.com/sususweet/midea_auto_cloud) | ✅ | ✅ | 走美的云 API 控制，支持空调、洗衣机、热水器等几十类设备 |

云端方案代价：控制经美的云中转，延迟 1~3 秒，依赖外网。家里断网时无法控制。

---

## 一、准备：美的美居账号

1. 手机下载 **美的美居** App（应用商店搜「美的美居」）。
2. 注册 / 登录账号（手机号即可，不需要短信验证码配 HA）。
3. 把空调绑定到账号下：
   - **美的空调**：在美的美居里添加设备。
   - **华凌空调**：两种情况都可以——
     - 直接在美的美居 App 里添加；
     - 或用 **美的美居 / 华凌智家** App 绑定后，设备会同步到同一美的账号下（实测华为智慧生活里通过「云云互联」绑定的华凌，也会同步出现在美的美居账号中）。
4. 确认 App 里能看到空调并能遥控，就可以进行下一步。

## 二、安装集成

### 方式 A：HACS（推荐）

1. HACS → 右上角 ⋮ → **自定义存储库**。
2. 仓库地址填 `https://github.com/sususweet/midea_auto_cloud`，类别选 **集成**。
3. 点添加，然后下载安装。
4. **重启 Home Assistant**。

### 方式 B：手动安装

1. 到 [Releases](https://github.com/sususweet/midea_auto_cloud/releases) 下载最新版（本教程验证版本：**v0.4.19**）。
2. 解压后把 `custom_components/midea_auto_cloud` 整个目录复制到 HA 配置目录的 `custom_components/` 下。
3. **重启 Home Assistant**。

> 首次添加该集成时，HA 会自动安装依赖 `lupa`，需要能访问 PyPI（HA 容器默认走 Docker 网络出网）。如果安装失败，检查 HA 容器的网络 / 代理。

## 三、添加集成（配置流程）

1. HA → **设置 → 设备与服务 → 添加集成**，搜索 `Midea Cloud Auto`（或「美的」）。
2. 按提示填写：
   - **账号**：美的美居手机号
   - **密码**：美的美居密码
   - **服务器**：国内选 **美的美居**；海外账号选 MSmartHome
3. 选择要导入的**家庭**（按家导入，账号下的设备会全部带进来）。
4. 集成会自动从美的云下载设备资源描述文件，显示「成功 N 个 / 跳过 M 个」属正常现象（已内置的类别会跳过）。
5. 确认后集成创建完成，账号下所有设备（空调 / 洗衣机 / 灯等）都会生成实体，**实体名称自动是中文的**。

## 四、实体说明（以空调为例）

每台空调会生成约 20 个实体：

| 类型 | 实体 | 说明 |
|------|------|------|
| climate | `climate.xxx_wen_kong_qi` | 温控器：开关 / 模式 / 温度 / 风速 / 摆风 / 预设 |
| binary_sensor | `设备状态` | 在线状态 |
| sensor | `当前温度` `室内湿度` `室外温度` `实时功率` `运行模式` | 部分型号不报室外温度 / 实时功率，显示 unknown 属正常 |
| sensor | `本月耗电量` `今年耗电量` | 云端统计的累计电量（kWh） |
| switch | `电辅热` `防直吹` `干燥防霉` `智控温` `随身感` `显示开关` `蜂鸣器` `新风除菌` | 按型号而定 |
| select | `上下摆动风向` `左右摆动风向` `无风感` | 按型号而定 |
| fan | `送风` | 送风专用实体 |

## 五、面板（Lovelace）配置示例

### 恒温器卡（最简）

```yaml
type: thermostat
entity: climate.fang_jian_kong_diao_wen_kong_qi
name: 华凌空调
features:
  - type: climate-hvac-modes
  - type: climate-fan-modes
    style: dropdown
  - type: climate-preset-modes
    style: dropdown
  - type: climate-swing-modes
    style: dropdown
```

### 万能遥控器卡（需要 [universal-remote-card](https://github.com/madmicio/universal-remote-card)）

完整示例见本仓库 [`examples/lovelace-remote.yaml`](examples/lovelace-remote.yaml)，包含：开关、制冷/制热/除湿/送风/自动、六档风速、常用温度一键直达。

### 近 30 天每日用电图（需要 [apexcharts-card](https://github.com/RomRider/apexcharts-card)）

```yaml
type: custom:apexcharts-card
graph_span: 30d
header:
  show: true
  title: 近30天每日用电（度）
apex_config:
  chart:
    height: 200
  legend:
    show: false
series:
  - entity: sensor.xxx_jin_nian_hao_dian_liang   # 换成你的"今年耗电量"实体
    name: 每日用电
    type: column
    float_precision: 2
    group_by:
      duration: 1d
      func: diff
```

## 六、与本地 midea_ac_lan 共存（重要）

如果你之前已经用 midea_ac_lan 接入了**美的**，现在又为了**华凌**装了云端集成，会出现同一台美的空调两套实体（本地一套 + 云端一套）。推荐处理方式：

- **控制用本地**（响应快、断网可用），云端那台美的设备禁用即可：
  设置 → 设备与服务 → midea_auto_cloud → 找到美的空调设备 → ⚙ → **启用/禁用**（禁用后其实体在重启 HA 后消失）。
- **华凌走云端**（本地接不进来，没办法）。
- 两套集成互不冲突，可长期共存。

## 七、常见问题

**Q：登录报错 / 要求短信验证码？**
连续多次登录失败会触发风控。等几分钟再试；正常一次登录不会要验证码。

**Q：添加集成时提示下载资源失败？**
HA 容器需要能访问美的云（`mp-prod.msmartlife.cn` 等）。检查容器 DNS 与外网连通性后重试。

**Q：设备列表里没有我的空调？**
确认它在美的美居 App 里可见。美的美居看不到的设备，HA 也拿不到。

**Q：控制延迟高？**
云端方案正常延迟 1~3 秒，属预期行为。

**Q：温度设定 17~30°C 之间有 0.5 步进？**
不同型号上下限不同，集成会按设备上报的能力自动限制。

**Q：「酷省电」「ECO」这些美的特有模式在哪？**
集成暴露 `preset_modes: eco / comfort / boost`，但**是否生效取决于型号**：部分设备只认部分预设（实测某美的挂机 eco/comfort 会被设备静默拒绝，boost 可用）。如果 preset 无效，可以用自动化等效模拟（例如「酷省电」≈ 制冷 26°C + 自动风）。

## 八、折腾实录：从「华凌死活加不进来」到找到解法

这一节是真实排查过程，供同样卡住的人参考。

**1）起点：美的早就接好了，华凌怎么都进不来**

家里美的空调是通过 `midea_ac_lan`（本地局域网）接入的，实体、遥控、耗电统计都正常，也早就在 HACS 里确认过是最新版。于是理所当然地想：华凌同属美的系，照着再添一次不就行了？

**2）第一次尝试：本地集成反复失败**

`midea_ac_lan` 添加华凌时始终拿不到设备。翻 issue 找到关键一条——**华凌新固件走 V3 协议，设备 token 无法从美居云端 API 拉取**（[issue #275](https://github.com/wuwentao/midea_ac_lan/issues/275)）。也就是说：不是配置没填对，是**这条路径对华灵根本不通**。手动填 IP 的临时办法（#488）也只能覆盖部分老设备，华凌依然不行。

> 顺带确认了另一件事：本地集成 `wuwentao/midea_ac_lan` 本身是新版本（2026.9.1），不是「版本旧所以不支持」，别浪费时间在升级上。

**3）转机：华为智慧生活里能看到两台空调**

在排查「华凌到底有没有被美的云端认可」时，想到手机上的 **华为智慧生活** App 里同时出现了两台空调——一台美的、一台华凌，而华凌是通过**美的美居账号云云互联同步**过来的。

这条线索价值极大：

- 说明这台华凌在**美的云端体系里是被认可的设备**，账号能看见、能下发指令；
- 既然云端能控，缺的就只是「局域网 token」；
- 而**云端控制本来也不需要 token** → 只要绕开局域网，走美的云，华凌就有救。

**4）方案选型：两条路**

| 路线 | 集成 | 判断 |
|------|------|------|
| 走美的云 | `sususweet/midea_auto_cloud` | ⭐ 约 290，2026-09 仍在更新，HACS Default 仓库可直接搜到；明确支持 `0xAC` 空调（华凌就是这类），还能顺带接洗衣机、热水器。代价：云端 1~3 秒延迟、依赖外网 |
| 走华为桥接 | `xiasi0/ha-huawei-smarthome` | 支持云云互联的第三方设备，但要华为账号登录（鸿蒙设备收挑战码）+ **每个型号一个适配器文件**，空调适配情况未知，折腾度高，放弃 |

选第一条（本教程已实测成功）。

> ⚠️ **第二条路（华为智慧生活桥接）我们没有实际去试**，只是评估后放弃，想折腾的可以自己去试，下面有说明。

**5）安装（HA 跑在路由器 Docker 里，没走 HACS）**

因为 HA 跑在 iStoreOS 路由器的 Docker 容器里、HACS 授权码要人肉确认，最后采用手动安装：

1. 在本机下载 GitHub 仓库 zipball，解开取 `custom_components/midea_auto_cloud`（82 个文件）打包成 tar.gz；
2. 通过 SSH 上传到路由器，解压进 HA 的配置目录 `custom_components/`；
3. `docker restart homeassistant`，HA 立刻识别到这个自定义集成（日志里 custom integration 的 WARNING 属正常）。

> 依赖 `lupa>=2.0` 是在「添加集成」那一步由 HA 自动 pip 装的；如果容器访问不了 PyPI 会 setup 失败，需要提前检查外网。

**6）登录：一次过**

配置流程填美居手机号 + 密码、服务器选「美的美居」，**直接登录成功，没触发短信验证码**。账号下只有一个家庭（实测这一步是**按家庭导入**，不支持挑单台设备）。

**7）结果：3 台设备全进来了，美的重复了**

账号下一共 3 台设备，全部导入：

- **房间空调 = 华凌 KFR-35G/N8HA1Ⅲ-P** ✅ 在线，制冷 24°C，耗电、室温传感器都有数；
- 美的空调 KFR-35G/N8KS1-1 —— 和本地集成重复了一份；
- 办公室电灯（云端显示 unavailable）。

重复的处理：**美的继续用本地那套**（响应快、断网可用），把云端那条美的设备**禁用**掉即可（设置 → 设备与服务 → midea_auto_cloud → 该设备 → 禁用；实体在重启 HA 后消失）。华凌只能走云端，本地集成接不进来。

**8）收尾：改名 + 面板**

- 设备名「房间空调」改成了「华凌空调」，实体名这个集成本身就是中文，不用手工翻译；
- 照着已有美的遥控页的同款结构，给华凌做了：总览按钮 → `#hl` 弹窗（恒温器 + 扫风 + 更多功能）→ 独立遥控页（万能遥控卡 + 30 天耗电柱状图）。

**一句话总结**：华凌卡住的根因是「本地 V3 协议拿不到 token」，突破口是「它在美的云端是被认可的」，所以解法不是继续折腾局域网协议，而是**换云端集成**。

**9）其实还有一条没去试的路：华为智慧生活桥接**

当时评估过、但没有实装的备选方案——**`xiasi0/ha-huawei-smarthome`**：

- 原理：把 Home Assistant 桥接到 **华为智慧生活**，它明确支持「云云互联的第三方设备」——而华凌正是通过云云互联同步进华为生态的；
- 优点：不依赖美的云，走华为链路；
- 代价 / 门槛（也是我们放弃的原因）：
  - 需要华为账号登录，鸿蒙设备还要收挑战码授权；
  - **每类设备需要一个适配器文件**（`custom_components/huawei_smarthome/device_adapters/`），没有适配器的型号就不会生成实体，华凌空调是否有现成适配器要自己翻仓库确认；
  - 社区规模比 midea_auto_cloud 小，踩坑没人带。

类似的还有 `Wangxiaokang666-666/ha_hwhomebridge`（方向相反：把 HA 设备同步**进**智慧生活，不适用于本需求）和 `xelhark/midea-cloud-relay`（把纯云端美的设备伪装成本地 V2 LAN 设备，更极客但更绕）。

如果你不想依赖美的云、且愿意自己写/找适配器，可以把这条路当作 Plan B。欢迎实测后在 issue 里补充结果。

## 九、参考与致谢

- [sususweet/midea_auto_cloud](https://github.com/sususweet/midea_auto_cloud) —— 本教程使用的云端集成
- [wuwentao/midea_ac_lan](https://github.com/wuwentao/midea_ac_lan) —— 本地局域网集成（美的推荐，华凌不支持）
- [midea_ac_lan issue #275](https://github.com/wuwentao/midea_ac_lan/issues/275) —— 华凌 V3 协议 token 问题的讨论
- [mac-zhou/midea-ac-py](https://github.com/mac-zhou/midea-ac-py) —— 早期美的云/本地协议实现

---

*本教程为个人实践记录，验证环境：Home Assistant 2026.9 + midea_auto_cloud v0.4.19 + 美的 KFR-35G/N8KS1-1 + 华凌 KFR-35G/N8HA1Ⅲ-P。*
