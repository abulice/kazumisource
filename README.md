# Kazumi 规则源

自用 Kazumi 规则集合 = **官方 KazumiRules 全量镜像 + 自研规则**。

## 为什么是全量镜像

Kazumi 的「规则仓库地址」**只能填一个 URL**，无法同时订阅官方和自己。
所以本仓库直接把官方 87 个规则文件 + `index.json` 全量并进来，再追加自研规则，
一个地址就能同时拿到官方全部规则和自己的规则。

## 订阅地址

```
https://raw.githubusercontent.com/abulice/kazumisource/main/index.json
```

国内访问不稳定时用 jsDelivr 镜像：

```
https://cdn.jsdelivr.net/gh/abulice/kazumisource@main/index.json
```

如果 App 只支持粘贴内容（导入规则），就把单个规则的 JSON 整个复制进去。

## 内容

- **88 个规则文件**：官方 `Predidit/KazumiRules` 全量 87 个 + 自研 `DM6900`
- **19 条 index 登记**：官方已验证的 18 条 + 自研 1 条

> 官方仓库里另有约 69 个 JSON 文件因失效/未验证未登记在 `index.json` 中，
> 文件仍保留，需要时可在 App 内手动指定。

## 自研 / 修复规则

| 规则 | 站点 | 状态 | 说明 |
|---|---|---|---|
| DM6900 | dm6900.com（樱花动漫） | ✅ 可用 | MacCMS + POST 搜索 + Base64 混淆 m3u8 |

### DM6900

- 搜索：`POST /index.php/vod/search2.html`，参数 `searchword`
- 列表：`//ul[@id='result']/li`，名字 `.//p[@class='name']`，链接 `.//a[@class='li-hv']`
- 分集：`//div[@id='stab_1_71']/ul`（每条 `<ul>` 是一条线路），剧集 `.//li/a`
- m3u8：播放页 `var paly_js = "<串>"`，按 `-`→`a`、`_`→`+`、`.`→`=` 还原 Base64；
  站点 `/ya.html?url=<串>&type=bjm3u8` 自行解密后跳 `jisuzyjx.com/play/?url=<明文m3u8>`，
  Kazumi WebView 从 `?url=` 参数捕获
- 该站 JSON API 全部关闭（`/api.php/provide/vod`、`/index.php/ajax/*` 均返回 System Error），故用 XPath 模式
- 实测《凡人修仙传》id=16998：线路1 200集 / 线路2 202集，m3u8 与 TS 分片均 200 可下载

## 已知失效站点

### yinghua（iyinghua.com）

官方 `yinghua.json` 已标记 `deprecated: true` 且从 index 中摘除。2026-11-16 实测：

| 路径 | 结果 |
|---|---|
| `/` | 200，UA 分流跳转页（约 7.5s） |
| `/pc/` | 200，仍是跳转页「正在跳转」 |
| `/app/` | 200，「APP 下载」页 |
| `/ios/` | 200，「iOS 下载」页 |
| `/search/凡人修仙传/` | **404**（旧规则 searchURL 失效） |

站点已重构成只推 App 的形态，web 搜索路径不存在。需重新逆向或改用 App 端接口。

## 同步官方更新

本仓库是官方快照，不会自动跟进。需要更新时：

```bash
cd /data/projects/kazumisource
git clone --depth 1 https://github.com/Predidit/KazumiRules.git /tmp/kzr
# 覆盖为官方最新文件（保留自研规则），再合并 index.json
```

或直接在 GitHub 网页上传官方新文件。

## 规则开发踩坑

- **XPath 相对路径必须带 `.` 前缀**：写 `.//li/a` 而不是 `//li/a`，
  否则 `//` 会从文档根匹配，把导航栏等无关链接也抓进来。
- `index.json` 是**元数据数组**，不是 url 映射表：
  `[{"name","version","useNativePlayer","antiCrawlerEnabled","author","lastUpdate"}]`
- 站点若有可用 JSON 接口，优先用 `api: "9"` + `searchMode: "api"`（JSONPath），比 XPath 稳。
- `useLegacyParser: false` 走 WebView 真实加载；`decodeVideoSource()`（`lib/utils/media.dart`）
  只从 URL query 参数正则抓 `http.*\.m3u8|mp4`，**本身不做 base64 解码**，解密靠站点自己完成。
