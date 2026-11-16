# Kazumi 规则源

自用 Kazumi 番剧规则集合。

## 用法

Kazumi → 设置 → 规则管理 → 导入 → 从 URL 导入：

```
https://raw.githubusercontent.com/abulice/kazumisource/main/index.json
```

国内访问 raw.githubusercontent.com 不稳定时可换 jsDelivr 镜像：

```
https://cdn.jsdelivr.net/gh/abulice/kazumisource@main/index.json
```

单条规则也可以直接导入：

```
https://raw.githubusercontent.com/abulice/kazumisource/main/DM6900.json
```

## 已有规则

| 规则 | 站点 | 说明 |
|---|---|---|
| DM6900 | dm6900.com（樱花动漫） | MacCMS 架构，POST 搜索，Base64 混淆 m3u8 由站点 `/ya.html` 自行解密 |

## 规则说明

### DM6900

- 搜索：`POST /index.php/vod/search2.html`，参数 `searchword`
- 搜索列表：`//ul[@id='result']/li`
- 分集：`//div[@id='stab_1_71']/ul`（每条 `<ul>` 是一条线路）
- 播放页 `/ya.html` 用 `-`→`a`、`_`→`+`、`.`→`=` 的规则还原 Base64，
  再跳转到 `jisuzyjx.com/play/?url=<明文 m3u8>`，Kazumi WebView 可自动捕获
- 该站 JSON API（`/api.php/provide/vod`、`/index.php/ajax/*`）均已关闭，故用 XPath 模式

## 已知问题

- `chapterResult` 必须用 `.//li/a`（带 `.` 前缀），否则会从文档根匹配到导航栏链接
- 站点有 Cloudflare，偶尔需要稍等重试
