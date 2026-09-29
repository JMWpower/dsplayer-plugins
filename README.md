# dsplayer-plugin

DsPlayer 插件官方市场仓库：`market.json` 为市场索引（在线安装入口），`packages/` 存放插件包，`icons/` 存放条目图标。

## 当前收录（15 个条目 = 环境 9 + 应用 6）

条目带 `category` 字段（2026-09-20 起）：`env`=环境（引擎/运行时，给壳子供能力，默认归类）；`app`=应用（面向完整使用场景：整合包、直播源包、服务包）。市场内按类别过滤，默认展示「环境」。

### 环境（env）

| 条目 | id | 版本 | type | 包 |
|---|---|---|---|---|
| 媒体代理服务 | `mediaProxy` | 1.1.1 | import（zip） | `packages/mediaProxy-1.1.1.zip` |
| Node.js 运行时 | `nodejs` | 1.0.0 | import（zip） | `packages/nodejs-1.0.0.zip` |
| PHP 运行时 | `php` | 1.3.1 | import（zip） | `packages/php-1.3.1.zip` |
| Python 爬虫引擎 | `py` | 1.1.3 | **apk**（系统安装） | `packages/DsPlayer-Python-plugin-1.1.3-arm64.apk` |
| MPV 播放内核 | `mpv` | 1.0.2 | import（apk 直装包可导入） | `packages/mpv-1.0.2.apk` |
| IJK 播放内核 | `ijk` | 1.0.1 | **apk**（桥接式插件必须系统安装，不支持 zip 导入） | `packages/ijk-1.0.1.apk` |
| FFmpeg 软解 | `ffmpeg` | 1.0.2 | **apk**（系统安装；需本体 v0.6.2+） | `packages/ffmpeg-1.0.2.apk` |
| QJS 爬虫引擎（dr2 + dr3 源） | `qjs` | 1.0.3 | import | `packages/qjs-1.0.3.apk` |
| AI 助手界面 | `agent` | 1.1.0 | import | `packages/agent-1.1.0.apk` |

### 应用（app）

| 条目 | id | 版本 | type | 包 |
|---|---|---|---|---|
| 插件整合包 | `bundle` | 1.1.0 | **apk**（系统安装） | `packages/bundle-1.1.0.apk` |
| IPTV 直播源（CCSH 采集） | `iptv-ccsh` | 1.1.0 | **live**（直播源包） | `packages/iptv-ccsh-1.1.0.json` |
| 洛雪同步 | `lx-sync` | 2.1.2 | **server**（服务包） | `packages/lx-sync-2.1.2.zip` |
| 弹幕 API 服务 | `danmu-api` | 1.0.0 | **server**（服务包） | `packages/danmu-1.0.0.zip` |
| 演示源包 | `demo-sources` | 1.1.3 | **source**（源码包） | `packages/demo-sources-1.1.3.zip` |
| IDM+ 下载器（1DM+） | `idmplus` | 18.2 | **apk**（系统安装；Release 分发） | `releases/download/idmplus-18.2/idmplus-18.2-CN.apk` |

### 引擎类条目版本要点

| 条目 | 当前版本 | 要点 |
|---|---|---|
| `mpv` | 1.0.2 | libmpv 重编入 DASH（MPD）demuxer（上游构建缺 libxml2 致 `ff_dash_demuxer` 未编入，DASH 源此前须降级 Exo）；内核 1.2.5 → 1.2.6 |
| `py` | 1.1.3 | 1.1.3 恢复插件存储读权限（Manifest 此前仅 INTERNET）：文件浏览器式本地源（资源管理.py 等）在**插件进程**内列目录/读文件必需——无权限时对他人属主文件 stat 全 EACCES、目录恒 0 项；补 READ + MANAGE_EXTERNAL_STORAGE + requestLegacyExternalStorage（**仅读**，写删约束仍由 fs_guard + 无 WRITE 承载），装机后需授予「文件/存储」权限（应用详情→权限）。1.1.2 并发与自愈：同源长任务占锁 10s 快速报忙；源实例创建移出全局锁；修复 unloadSource 逐出通道静默空转；配套本体超时自动换实例 + 分发池 8 线程。1.1.1 修 BaseSpider 构造期崩溃（`__init__` 不再调用可被子类重写的 `self.log`，改模块级 `_log` 直调——hipy 源「资源管理」AttributeError 实锤，T3 官方基类对齐，上游 drpy-node 已同步修）。1.1.0 大响应 callBigFile 通道 + 抗杀保活开关（插件页 py 卡片，BIND_IMPORTANT，默认关） |同源长任务占锁时后续调用 10s 快速报忙（不再 30s 长队堆积）；源实例创建移出全局锁（新源首次加载不堵其他源）；修复 unloadSource 逐出通道自设计起静默空转；配套本体 v0.7.2+ debug（超时自动换实例 + 分发池 8 线程）后一个源失控不拖累其他源、失控源超时后秒级自愈。1.1.1 修 BaseSpider 构造期崩溃——`__init__` 不再调用可被子类重写的 `self.log`（改模块级 `_log` 直调）：源重写 log 且在 `super().__init__()` 之后才赋值其引用的属性时（hipy 源「资源管理」实锤 AttributeError），构造中断实例残废；T3 官方基类构造期不调 log，兼容性对齐，上游 drpy-node 已同步修。1.1.0 大响应 callBigFile 通道（AIDL 追加新方法，engine 1.1.0）+ 抗杀保活开关（插件页 py 卡片，BIND_IMPORTANT，默认关） |
| `qjs` | 1.0.3 | so 升级（qjs_ultra build-20260928）：**cheerio 补齐 :gt/:lt 切片**（选择器伪类 + 链式方法，外部贡献），drpy2/drpy3 源 HTML 解析选择器更全。ABI 与 1.0.2 一致（66 导出闸门校验），旧本体兼容。此前 1.0.2 drpy3 源运行时并入本插件（fjs 退役下架）+ 原生 WebAssembly（wasm3）+ 墙钟超时中断/结构化错误/值转换护栏；1.0.1 根治跨 isolate SIGABRT（回调 per-context 注册） |
| `bundle` | 1.1.0 | 移除 fjs 子插件（drpy3 并入 qjs 1.0.2），全家桶现为 MPV/Python/QJS/Agent 四插件；此前 1.0.9 agent 1.0.0→1.1.0（NextChat v2.15.8 + injectCompat） |
| `ijk` | 1.0.1 | 1.0.0 首版（DexClassLoader 桥接，CarGuo 修正版 ijkplayer，HTTPS/16K page size）真机播放/切集/连播/三内核切换全通；1.0.1 修 UA 透传——IJK n4.3 的 headers 字典不生效到 HTTP 请求头（部分 CDN/防盗链源拒默认 UA 报 400），UA 改走 user_agent 协议级 option 直达。**必须 APK 直装**（files zip 导入会丢 dex 致本体探测失效） |
| `ffmpeg` | 1.0.2 | 1.0.2 修复软解画面纯色闪烁（Flutter SurfaceProducer 尺寸协商，需搭配最新本体）；1.0.1 补载 NDK C++ 运行时 libc++_shared.so（真机实锤：FongMi 编的 libavcodec 等动态依赖它，缺失时 dlopen 直接失败）；1.0.0 首版——FongMi/media fork（release-1.11.0-fongmi）编出的 decoder_ffmpeg so 载体：音频软解（AC3/EAC3/DTS 全家/TrueHD/Atmos 等）+ 视频软解（H.264/H.265/AV1/VP9/MPEG-4/AVS2/AVS3，Dolby Vision 基础层映射），为 Exo 内核补第三层解码兜底（硬解不支持自动回落，硬解可用时零开销）。**需本体 v0.6.2+**（media3 切 fork 版 + FfmpegDecoderLoader 加载链），**必须 APK 直装** |

各包完整变更说明见 `market.json` 条目的 `changelog` 字段（DsPlayer 详情弹层直接展示）。

图标在 `icons/`（与条目 `icon` 字段对应；fjs.png 已随条目下架弃用，保留留档）。`iptv.png` 为已弃用的旧版图标（被 `iptv2.png` 取代，保留留档）。

弹幕 API 服务配套用法：服务启动后，DsPlayer 设置 → 播放器 → 弹幕接口 填 `http://127.0.0.1:9321`，播放无自带弹幕的影片即自动按标题匹配（兼容弹弹play 协议）。

## 条目形态（type）与安装语义

| type | 包体 | 安装动作 | 典型条目 |
|---|---|---|---|
| `import` | zip / apk | 应用内静默导入（组件落应用内目录） | 引擎/运行时类 |
| `apk` | apk | 跳系统安装器直装 | py、bundle、ijk、ffmpeg、idmplus |
| `live` | JSON（`{"lives":[{name,url,ua,epg}]}`） | 写入直播配置并启用，切直播页生效；**订阅制**（内容指向外部地址时随源自动更新） | iptv-ccsh |
| `server` | zip（根部须有 `server.json` manifest：`serviceName/workDir/entry/port/healthType/desc`） | 解压到 `sdcard/dsplayer/server/node/`（覆盖式，数据目录保留）+ **自动创建服务配置**（nodejs 运行时启动；服务 id 约定 `svc-mkt-<条目id>`，已存在跳过） | lx-sync、danmu-api |
| `source` | zip（根部可选 `source.json` manifest：`{"dirs":["dr3","js"]}` 目录白名单，缺省全解压） | 解压到 `sdcard/dsplayer/spider/`——包内顶层目录与本地源扫描目录（`dr2`/`dr3`/`hipy`/`js`）同名直落位，覆盖式不影响包外文件；进对应本地源页自动扫描入库；已装态存 App 安装记录（更新 = 市场版本对比后重装）。**zip 文件名必须 UTF-8 编码**（7-Zip 默认按本地代码页打包中文名会乱码，用 Python zipfile/UTF-8 工具打包） | demo-sources |

`server` 包要求设备已装 `nodejs` 运行时插件；manifest 字段由 DsPlayer 市场安装器消费（见 DsPlayer 仓库 `MarketManager.installServerPackage`）。

大体积 APK（50MB 量级及以上）可不进 `packages/`，改发 GitHub Release 分发：tag 约定 `<id>-<version>`（升版本发新 tag，同名资产不覆盖——代理 CDN 缓存纪律同 packages/），asset 文件名避开 `+`/中文等需 URL 编码字符，条目 `url` 写**绝对直链**——客户端 `resolveMarketUrl` 对 `https://` 原样直通、`applyGithubProxy` 对 github.com 域自动套加速代理。首个样例 `idmplus`。

## 使用

DsPlayer → 插件中心 → 市场 → 添加市场，填入本仓库索引直链：

```
https://raw.githubusercontent.com/hjdhnx/dsplayer-plugin/main/market.json
```

### 镜像与加速

GitHub 直连不畅时，任选其一（DsPlayer 内「管理市场 → GitHub 加速代理」已内置前两个，无需手动拼地址）：

- jsDelivr CDN：`https://cdn.jsdelivr.net/gh/hjdhnx/dsplayer-plugin@main/market.json`
  - ⚠️ jsDelivr 对分支引用有 CDN 缓存（索引推送后可能滞后数小时）。索引更新后可主动刷新缓存：
    `https://purge.jsdelivr.net/gh/hjdhnx/dsplayer-plugin@main/market.json`（浏览器访问一次即可）
- gh-proxy 类前缀代理（实测推荐序，2026-09-17）：
  1. `https://gh-proxy.com/` —— 最快最稳，DsPlayer 内置默认
  2. `https://gh-proxy.playdreamer.cn/` —— 稳定
  3. `https://github.catvod.com/` —— 偶发 502/截断，备选
- 用法：前缀 + 完整原始 URL，如 `https://gh-proxy.com/https://raw.githubusercontent.com/hjdhnx/dsplayer-plugin/main/market.json`

## 索引格式

见 `market.json` 与 DsPlayer 仓库 `docs/plugin/PLUGIN-MARKET-DESIGN.md` §二：

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | ✓ | 条目唯一身份：binary/runtime 插件 = 包内 plugin.json 的 `name`；内置引擎 = `mpv/py/qjs/agent`；应用类自定（iptv-ccsh/lx-sync/danmu-api/demo-sources） |
| `name` / `version` / `url` | ✓ | 展示名 / 语义化版本 / 包地址（绝对直链或相对本索引的路径） |
| `type` | ✓ | `import` / `apk` / `live` / `server`（语义见上表） |
| `category` | | `env`（默认）/ `app`；未声明归 env |
| `icon` | | 图标地址（绝对 URL 或相对本索引的路径），未声明回落首字母占位 |
| `size` / `author` / `desc` / `tags` / `changelog` | | 展示元数据 |
| `md5` | | 包校验和（32 位 hex）；**声明即强制校验**，不符拒装防篡改。官方 12 包全量声明，可用 `md5sum packages/<包名>` 复核 |
| `minApp` | | 可选；要求的最低 DsPlayer 版本，不满足时安装按钮置灰 |
| `pkg` | | 可选；**type=apk 专属**——应用包名。声明后客户端走通用包探测判定已安装（第三方插件零壳子改动接入的关键：不在 DsPlayer 内置插件注册表的 apk 条目必须声明，否则市场 UI 恒判「未安装」） |

## 发布约定（2026-09-20 起执行）

1. **同名包绝不重传**：内容有任何变化一律升版本号并换新文件名（如 `lx-sync-2.1.2.zip`），代理/CDN 层对同名文件的缓存会导致客户端「md5 校验不符」假失败（lx-sync 实锤）。旧版本包删除（git 历史留档）。
2. `md5` / `size` / `version` / `changelog` 与包严格同步；顶层 `updatedAt` 每次发布刷新。
3. `server` 条目包内 `server.json` 为安装指令，DsPlayer 安装时跳过落盘；`live` 条目包体即数据。
4. 图标换图时**换文件名**（如 `iptv.png`→`iptv2.png`），客户端图片磁盘缓存按 URL 键控。

## 自建市场

任意能放静态文件的地址（GitHub 仓库 / 对象存储 / 本地 sdcard）都可作市场：一份索引 JSON + 包文件即可。第三方条目的 `id` 须与包内 `plugin.json` 身份一致（`live`/`server` 条目除外：`live` 无插件身份，`server` 以 manifest 建服务），否则已装判定不闭环。
