<img src="icon.png" width="96" align="right" alt="iOS Lite">

# iOS 瘦身 · iOS Lite

[![validate](https://github.com/k6nfmm7dbr-commits/ios-lite/actions/workflows/validate.yml/badge.svg)](https://github.com/k6nfmm7dbr-commits/ios-lite/actions/workflows/validate.yml)
![clients](https://img.shields.io/badge/clients-Egern%20%7C%20Surge%20%7C%20Loon%20%7C%20Stash%20%7C%20Shadowrocket-blue)
![license](https://img.shields.io/badge/license-MIT-green)

全系统通用的去广告 / 隐私模块：跨 App 拦截**广告 SDK、统计、归因、崩溃上报**域名，
减少开屏广告等待、广告加载掉帧和 App 启动时的后台抢网。

支持 **Egern**（原生 YAML 模块）、**Surge / Stash / Shadowrocket**（`.sgmodule`）、
**Loon**（`.plugin`）。零 MITM、零证书、零脚本。

---

## ⚠️ 先说清楚：这个模块救不了卡顿

如果你手机卡，**不要指望装个模块就好**。代理层能控制的只有网络请求，
而卡顿的成因几乎都在别处：SoC 降频、内存不足、电池老化。

本模块真正能改善的是**体感卡顿**，不是性能：

| 你感受到的 | 模块能不能帮上 |
|---|---|
| 点开 App 先看 3~5 秒开屏广告 | ✅ 能 —— 广告加载不出来，直接跳过 |
| 刷信息流时广告插入处的掉帧 | ✅ 一部分 |
| App 启动时"愣一下" | ⚠️ 一点点（统计 SDK 不再抢网络） |
| 系统整体滑动掉帧、动画卡顿 | ❌ **完全不能** |
| 手机发烫后变慢 | ❌ **完全不能** |

**真正该做的事在下面第二节，请先做那个。**

---

## 一、诊断：以一台真实设备为例

下面是一台 **iPhone 13（iPhone14,5）/ iOS 27.2** 的实测状态，正好是"有点卡顿"的典型：

| 指标 | 实测值 | 判断 |
|---|---|---|
| 机型 | iPhone 13 | 2021 年 A15 |
| 系统 | iOS 27.2 | 最新版 |
| **热状态** | **`serious`** | 🔴 **系统正在降频** |
| 可用内存 | 3.58 GB | 🟡 4GB 机型，iOS 27 下偏紧 |
| 存储剩余 | 151 GB / 238 GB（用了 36%） | 🟢 正常，**不是卡顿原因** |
| 开机时长 | **10.7 天** | 🟡 建议重启 |
| 电池 | 65%，充电中 | 🟡 充电本身在发热 |

**结论**：存储排除了，问题指向**热节流 + 内存压力**，而不是网络。
装任何代理模块都不会改变这两项。

### 热状态 `serious` 是什么意思

iOS 的散热状态分四级：`nominal` → `fair` → **`serious`** → `critical`。
到 `serious` 时系统已经**明显压低 CPU / GPU 频率**来降温 —— 这就是"突然变卡"的直接原因。
而当时设备**正在充电**，充电本身就是主要热源之一。

---

## 二、真正该做的事（按优先级）

### P0 · 解决发热 —— 这一条顶其他所有

- **充电时别重度使用**。实测就是充电中触发 `serious`。边充边玩是 A15 降频最常见的场景。
- **摘掉厚手机壳**，尤其是充电和使用时。很多壳把热量闷在里面。
- **避免阳光直射 / 车内 / 被子上**使用。
- **关掉「优化电池充电」以外的边充边用习惯**：想快充就放着别动。
- 如果装了 MagSafe 无线充：无线充电发热比有线大得多，边充边用更容易降频。

### P0 · 查电池健康（iPhone 13 已 5 年机龄，这条极可能是主因）

> 设置 → 电池 → 电池健康与充电

看两个地方：

- **最大容量**：低于 **80%** 就该换电池了。
- **峰值性能容量**：如果显示「**已启用性能管理**」或提到「因电池无法提供必要的峰值功率…」
  —— 那 iOS 就**在主动限制你的 CPU 频率**，卡顿是必然的。

> ⚠️ 这一项 API 不对外开放，所以上面那张表里我读不到它，**只能你自己去看**。
> 如果是这个原因，**换块电池是唯一有效的解**，比任何软件设置都管用。

### P1 · 重启

实测开机 **10.7 天**。长时间不重启，内存碎片、后台残留、系统服务缓存都会累积。
**先重启一次，观察半天** —— 这一步零成本，经常直接解决问题。

### P1 · 接受 4GB 内存的现实

iPhone 13 是 **4GB RAM**，而 iOS 27 的基线比发布时重了不少。
能做的：

- **别留一堆后台 App**：iOS 会杀后台来回收内存，被杀的 App 重新加载 = 你感受到的"卡"。
- **关掉「后台 App 刷新」**：设置 → 通用 → 后台 App 刷新 → 全部关掉。
  微信 / QQ 的消息推送走 APNs，**关掉不影响收消息**。
- 少开大游戏 / 大 App 之间来回切。

### P2 · 减轻系统动画负担

> 设置 → 辅助功能 → 动态效果

- 打开「**减弱动态效果**」
- 打开「**减弱透明度**」（降低毛玻璃渲染开销，老机型效果明显）

> 设置 → 辅助功能 → 显示与文字大小 → 打开「**降低白点值**」
> —— 能降低屏幕亮度相关的功耗与发热。

### P2 · 顺手清理

- 设置 → 通用 → iPhone 储存空间 → 卸载长期不用的 App
- 设置 → Safari → 清除历史记录与网站数据
- 设置 → 通用 → 后台 App 刷新 里，只留必要的

### P2 · 代理层自己也要瘦身

这一条跟本模块系列直接相关：**代理 App 是有成本的**。

- **MITM 每解密一条连接都要算一次 CPU**。对已经降频的机器，这是纯负担。
  → 我前面做的 `wechat-lite-plus` 和 `douyin-lite-plus` **都需要 MITM**，
  **不要同时开**，更不要为了小收益长期开着。
- **模块不是越多越好**：每个模块的规则都要参与匹配。用完的模块就关掉。
- **定期看一眼 MITM hostname 列表**，把不再需要的域删掉。
- 如果只是想要去广告、不在乎个别接口级的拦截，**优先用各家的"基础版"**（零 MITM）。

---

## 三、模块做什么

只有一件事：**跨 App 拦掉广告 SDK 和统计 SDK 的域名**。零 MITM、零证书、零脚本。

### 为什么这样设计

- **不做 MITM**：解密流量要给已经降频的 CPU 加负担，与"让手机更顺"的目标背道而驰。
  规则匹配是纯字符串比对，几乎零成本。
- **不做脚本 / jq**：JS 运行时本身就是开销，而且规则会随各家接口变化失效。
- **不拦推送服务**（`getui.com` / `jpush.cn` / `igexin.com`）：
  这些域名不只做统计，还承担推送。拦了整个 App 的推送就没了。
- **不拦系统服务域名**：会出各种玄学问题。

### 拦截清单

| 分类 | 条目 |
|---|---|
| **字节穿山甲 / Pangle** | `pangle.cn`、`pangle.io`、`pangolin`、`pglstatp-toutiao` |
| **腾讯优量汇 / 广点通** | `pgdt.gtimg.cn`、`gdt.qq.com`、`l.qq.com`、`sdk.e.qq.com` |
| **百度联盟** | `mobads.baidu.com` |
| **阿里妈妈 / 淘宝联盟** | `afpapi.alimama.com`、`wgo.mmstat.com` |
| **国外广告** | `applovin`、`adsystem`、`doubleclick.` |
| **统计 / 归因 / 崩溃** | `umeng`、`appsflyer`、`adjust.`、`crashlytics`、`bugly.qq.com` |

共 19 条规则。这些是**国内 App 里铺得最广的几家**，
拦掉它们能覆盖相当大一部分开屏广告和信息流广告。

---

## 四、安装

| 客户端 | 导入链接 |
|---|---|
| **Egern** | [ios-lite.yaml](https://raw.githubusercontent.com/k6nfmm7dbr-commits/ios-lite/main/egern/ios-lite.yaml) |
| **Surge / Stash / Shadowrocket** | [ios-lite.sgmodule](https://raw.githubusercontent.com/k6nfmm7dbr-commits/ios-lite/main/surge/ios-lite.sgmodule) |
| **Loon** | [ios-lite.plugin](https://raw.githubusercontent.com/k6nfmm7dbr-commits/ios-lite/main/loon/ios-lite.plugin) |

- **Egern**：工具 → 模块 → 右上角 `+` → 粘贴上面的链接
- **Surge**：首页 → 模块 → 安装新模块 → 粘贴链接
- **Loon**：配置 → 插件 → `+` → 粘贴链接
- **不想托管任何文件**：把 [`egern/snippet-profile.yaml`](egern/snippet-profile.yaml) 里的
  `rules` 直接粘进 Egern 主配置 `Profile.yaml` 即可

### 进阶：外挂社区规则集

19 条规则只是"高价值精选"。想要更彻底的全网追踪拦截，别手抄几千条域名 ——
直接让 Egern 引用远程规则集，由它自己下载维护。`snippet-profile.yaml` 末尾给了现成写法：

```yaml
rules:
  - rule_set:
      match: "https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Surge/Advertising/Advertising.list"
      policy: REJECT
      update_interval: 86400
  - rule_set:
      match: "https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Surge/EasyPrivacy/EasyPrivacy_Domain.list"
      policy: REJECT
      update_interval: 86400
```

> `EasyPrivacy` 有几万条、覆盖面极大，**有可能误伤个别 App 的正常功能**。
> 建议先只开 `Advertising` 观察一段时间再加。

---

## 五、验证 & 风险

### 验证

1. 请求记录 → 打开任意一个带开屏广告的 App → 看 `pglstatp-toutiao.com`、
   `mobads.baidu.com` 之类是否显示 `REJECT`；
2. 主观体感：开屏广告是不是少了 / 短了。

如果规则没生效，**九成是顺序问题** —— 被主配置里更靠前的宽泛直连规则抢先命中了。

### 风险

| 风险 | 说明 | 缓解 |
|---|---|---|
| 规则不生效 | 被更靠前的宽泛规则抢先命中 | 用请求记录确认，调整顺序 |
| 关键词匹配面较广 | `umeng`、`adjust.`、`pangolin` 是关键词匹配 | 出问题就注释掉对应那行，改成精确域名 |
| 个别 App 功能异常 | 少数 App 在统计上报失败时行为异常（罕见） | 关掉模块开关即可完全恢复 |
| 归因 SDK 被拦 | 极少数 App 的「邀请码 / 渠道统计」功能可能失效 | 同上 |
| `l.qq.com` 等域名误伤 | 腾讯的广告点击域，理论上也可能被非广告业务使用 | 注释掉那行 |

**回滚**：关掉模块开关即可，没有改任何持久状态。

---

## 六、域名清单来源

| 来源 | 说明 |
|---|---|
| [blackmatrix7 / Advertising](https://github.com/blackmatrix7/ios_rule_script) | `umeng`、`appsflyer`、`adjust.`、`crashlytics`、`applovin`、`adsystem`、`doubleclick.`、`mobads.baidu.com`、`afpapi.alimama.com`、`wgo.mmstat.com` |
| [blackmatrix7 / EasyPrivacy](https://github.com/blackmatrix7/ios_rule_script) | 交叉核对追踪器域名 |
| [LoonKissSurge / 广告平台拦截器.beta](https://github.com/QingRex/LoonKissSurge) | `pangle.io`、`pangolin`、`pglstatp-toutiao` |
| [blackmatrix7 / ByteDance](https://github.com/blackmatrix7/ios_rule_script) | `pangle.cn` 等字节系广告域 |

模块里的每一条都能在上述规则集里找到对应，没有凭空写的域名。

---

## 附：这个模块系列

同系列（结构、规范、CI 完全一致）：

- [wechat-lite](https://github.com/k6nfmm7dbr-commits/wechat-lite) —— 微信去广告 / 降内存
- [douyin-lite](https://github.com/k6nfmm7dbr-commits/douyin-lite) —— 抖音去广告 / 降内存
- **ios-lite** —— 全系统通用广告 / 追踪拦截（本仓库）

> 再提醒一次：这三个都是"少下载 → 少解码 → 少缓存"这条链路上的优化。
> 手机发烫降频、内存不足、电池老化，它们一个都解决不了。

---

## License

[MIT](LICENSE)
