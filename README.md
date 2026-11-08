# A股投研雷达 · 分发包

把这套本地投研控制台装到另一台 Windows 电脑上：**复制两样东西** —— 发布压缩包解压出来的文件夹 + `PROMPT.md` 里那段提示词。

装好后双击 `console\start-console.bat`，浏览器打开 `http://127.0.0.1:8080/`，即可看到与源环境一致的五个看板、技巧工坊与任务控制台。

---

## 你只需要做两件事

1. 把发布压缩包解压到本机任意目录（例如 `C:\radar`），**路径不要带空格更省心**。
2. 打开 `PROMPT.md`，把分隔线里的整段文字复制进 WorkBuddy 并发送。

WorkBuddy 会完成环境预检、建虚拟环境、装依赖、写入示例数据、拉起服务并自检。你不需要手工敲任何安装命令。

> 如果不想用 WorkBuddy，也可以直接命令行运行：`python deploy\bootstrap.py`（需已装 Python ≥ 3.10）。

---

## 双击启动 / 停止（重要）

| 动作 | 怎么做 |
| --- | --- |
| 启动 | 双击 `console\start-console.bat`，等端口就绪后浏览器自动打开 |
| 登录 | **已取消**：本机打开即用，无账号密码；服务只监听 `127.0.0.1` |
| 停止 | 关闭标题为 **`RadarConsole`** 的服务窗口（或在其中按 `Ctrl+C`）；另一个 launcher 窗口可直接关掉 |
| 再次使用 | 重新双击一次 bat |

首版**不做开机自启、不注册系统服务**：服务不随 WorkBuddy 会话消失，但也只在 `RadarConsole` 窗口开着的时候运行。bat 为纯 ASCII 英文（cmd 按当前代码页解析 .bat，中文叠加 `chcp 65001` 会把中文片段当命令执行，导致服务根本不启动），会先起服务、等 8080 真正监听后再打开浏览器，并自动为本地回环设置 `NO_PROXY` 绕过代理。

---

## 首次打开看到的是示例数据

包里带一份脱敏示例快照（7 只中性演示标的 + 市场级指数/板块数据），只用于让五个看板首次打开不空屏，**不含分发者的任何个人自选股或素材图片**。

在控制台运行任一任务后，脚本会抓取当日真实行情，自动生成新快照并覆盖示例文件（按文件时间取最新）。示例文件内也标注了「运行任务后自动覆盖」。

---

## 目录结构

```
radar-pack/
├── PROMPT.md                 复制到 WorkBuddy 的提示词
├── README.md                 本文件
├── requirements.txt          依赖（akshare / pandas / numpy / openpyxl 等）
├── manifest.json             版本 + 文件清单 + SHA256
├── console/                  页面与本地服务
│   ├── server.py             单进程 HTTP 服务（默认 127.0.0.1:8080）
│   ├── start-console.bat     双击启动
│   ├── 投研雷达控制台.html   任务控制台
│   ├── 选股.html / workshop.html / theme.css
├── scripts/                  5 个任务脚本 + 指标算法
├── config/                   指标注册表 / 规则 / 素材股票池（种子）
├── demo/                     脱敏示例快照模板
├── deploy/                   bootstrap.py（一键部署）+ verify_pack.py（交付前校验）
├── skills/radar-deploy/      WorkBuddy 部署 Skill
└── data/                     安装后自动生成（运行数据 / config / Materials 素材目录）
```

安装后会自动生成 `data\`（所有运行数据与配置）与 `.venv\`（独立 Python 环境），二者都不属于分发内容。

---

## 五个看板与任务

| 看板 | 任务脚本 | 说明 |
| --- | --- | --- |
| 盘前热点 | `daily_hot.py` | 指数/广度/板块/外围/政策/风险 |
| 自动选股 | `stock_pool.py` | 规则过滤 + 综合评分 + Excel 下载 |
| 盘中自选监控 | `watch_monitor.py` | 自选行情、量比、异动与信号 |
| 盘后趋势 | `trend_report.py` | 均线/量能/MACD/周线/形态评分 |
| 复盘研判 | `review_report.py` | 汇总前四者生成复盘与明日研判 |

`review_report` 运行前会检查「盘前热点」是否为当日收盘后快照，若陈旧会自动刷新一次。

---

## 配置与个性化

- `data\config\watch_stock.csv`：自选股清单（`code,name` 每行一只），也可在控制台左侧「自选股票清单」里直接编辑保存。
- `data\config\indicators.json`：技巧工坊的指标注册表（评级/权重/启停/作用看板）。
- `data\Materials\`：技巧素材图片，可在技巧工坊页面上传、刷新、打开文件夹。

---

## 升级

只覆盖 `console\` 与 `scripts\` 两个目录。**绝不覆盖** `data\` 与 `Materials\` —— 那里是运行数据和你自己的素材。升级前建议先备份整个 `data\`。

---

## 安全与限制

- 服务默认只绑定 `127.0.0.1`，不要改成 `0.0.0.0` 后暴露到公网；如需远程访问请自行加 TLS 反向代理。
- 本机若设了 `HTTP_PROXY` / `HTTPS_PROXY`：bat 与 bootstrap 已自动绕过 `127.0.0.1`；镜像站被代理拦截时**不要加 `--mirror`**，用默认官方 PyPI。
- 默认免登录，不生成任何密码文件；将来若要公网部署，把 `start-console.bat` 里的 `AUTH_REQUIRED` 设为 `1` 恢复登录，并配置强密码 + TLS 反向代理。
- 行情来自 akshare 的公开接口，需要联网；非交易时段或网络受限时可能抓不到数据。
- 首版仅支持 Windows（「打开文件夹」「一键启动 WorkBuddy」依赖 Windows API）。
- 所有内容仅为公开市场数据的统计整理，不构成投资建议。
- 第三方依赖（akshare / pandas / numpy / openpyxl 等）各自遵循其开源许可，行情数据版权归相应数据源所有。

更完整的部署/排障说明见 `skills\radar-deploy\references\部署说明.md`。

---

## 发布者须知：只打包 manifest 白名单（重要）

运行过一次的目录里会生成 `.venv\` 与 `data\`。**这两项绝不能随包发布**：

- `.venv\pyvenv.cfg` 写死了源机器的 Python 绝对路径，换一台机器就失效；而 `start-console.bat` 会优先使用这个 `.venv`，带着它发布等于让接收者双击即失败。
- `data\` 是本机运行数据与自选清单；带着它发布，接收者会看到旧日期数据，且 `bootstrap.py` 会因为目录非空而跳过示例快照写入。

发布包应包含 `manifest.json` 的 `files` 字段列出的 34 个文件，**外加 `manifest.json` 自身**（它是清单文件，不列在自己的 `files` 里，否则哈希会自引用循环）。可据此生成压缩包：

```python
# 在 radar-pack 目录执行
import json, pathlib, zipfile
m = json.loads(pathlib.Path("manifest.json").read_text(encoding="utf-8"))
with zipfile.ZipFile("../radar-pack-" + m["version"] + "-win.zip", "w",
                     zipfile.ZIP_DEFLATED) as z:
    for it in m["files"]:
        z.write(it["path"], "radar-pack/" + it["path"])
    z.write("manifest.json", "radar-pack/manifest.json")
```

注意：`deploy\verify_pack.py` 会主动跳过 `.venv\` 与 `data\` 目录，因此它「校验通过」只代表 34 个源码文件干净，**不代表整个运行目录可以直接压缩发布**。打包后请在一台干净机器（或空目录解压）按上面「你只需要做两件事」实测一次：依赖安装 → 示例快照写入 → 双击 bat → 浏览器打开首页。
