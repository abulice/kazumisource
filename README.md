# Kazumi 规则源

自用 Kazumi 番剧规则集合。与官方仓库**双订阅并存**，各管各的。

## 订阅方式

Kazumi → 设置 → 规则管理 → 规则仓库地址，填**两个**（逗号或换行分隔，按你 App 版本的支持方式）：

```
https://raw.githubusercontent.com/Predidit/KazumiRules/main/index.json
https://raw.githubusercontent.com/abulice/kazumisource/main/index.json
```

- 官方仓库天天更新，会自动跟进
- 本仓库只放自研/修复的规则，不会被上游覆盖

国内访问 raw.githubusercontent.com 不稳定时，本仓库可用 jsDelivr 镜像：

```
https://cdn.jsdelivr.net/gh/abulice/kazumisource@main/index.json
```

也可以直接粘单条规则 JSON（导入规则 → 粘贴内容）。

## 已有规则

| 规则 | 站点 | 状态 | 说明 |
|---|---|---|---|
| DM6900 | dm6900.com（樱花动漫） | ✅ 可用 | MacCMS + POST 搜索 + Base64 混淆 m3u8 |

## 规则说明

### DM6900（可用）

- 搜索：`POST /index.php/vod/search2.html`，参数 `searchword`
- 列表：`//ul[@id='result']/li`，名字 `.//p[@class='name']`，链接 `.//a[@class='li-hv']`
- 分集：`//div[@id='stab_1_71']/ul`（每条 `<ul>` 是一条线路），剧集 `.//li/a`
- m3u8：播放页 `var paly_js = "<串>"`，按 `-`→`a`、`_`→`+`、`.`→`=` 还原 Base64；
  站点 `/ya.html?url=<串>&type=bjm3u8` 自行解密后跳 `jisuzyjx.com/play/?url=<明文m3u8>`，
  Kazumi WebView 从 `?url=` 参数捕获
- 该站 JSON API（`/api.php/provide/vod`、`/index.php/ajax/*`）全部关闭，故必须用 XPath 模式
- 实测《凡人修仙传》id=16998：线路1 200集 / 线路2 202集，m3u8 与 TS 分片均 200 可下载

## 待修复

### yinghua（站点改版，规则失效）

官方 `Predidit/KazumiRules/yinghua.json` 已被标记 `deprecated: true` 并从上游 `index.json` 中摘除。
2026-11-16 实测 `iyinghua.com`：

| 路径 | 结果 |
|---|---|
| `/` | 200，UA 分流跳转页（约 7.5s） |
| `/pc/` | 200，仍是跳转页「正在跳转」 |
| `/app/` | 200，「APP 下载」页 |
| `/ios/` | 200，「iOS 下载」页 |
| `/search/凡人修仙传/` | **404**（官方旧规则的 searchURL 已失效） |

结论：站点已重构成「只推 App」的形态，旧的 web 搜索路径不存在。
需要重新逆向或改用 App 端接口，暂不纳入可用规则。

## 规则开发踩坑

- **XPath 相对路径必须带 `.` 前缀**：写 `.//li/a` 而不是 `//li/a`，
  否则 `//` 会从文档根匹配，把导航栏等无关链接也抓进来。
- `index.json` 是**元数据数组**，不是 url 映射表：
  `[{"name","version","useNativePlayer","antiCrawlerEnabled","author","lastUpdate"}]`
- 站点若有可用 JSON 接口，优先用 `api: "9"` + `searchMode: "api"`（JSONPath），比 XPath 稳。
- `useLegacyParser: false` 走 WebView 真实加载；`decodeVideoSource()`（`lib/utils/media.dart`）
  只从 URL query 参数里正则抓 `http.*\.m3u8|mp4`，**本身不做 base64 解码**，解密要靠站点自己完成。
