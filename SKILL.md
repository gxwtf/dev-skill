---
name: gxwtf-dev
version: 1.0.0
description: 广学五题坊（gxwtf）开发规范与项目地图。当需要在 gxwtf 组织下的仓库里改代码、建新站、接单点登录、部署上线、写文档，或回答「这个项目怎么跑 / 部署在哪 / 用什么规范」时使用。包含技术栈基线、SSO 契约、UI 与提交规范、部署流水线与常见坑。
---

# 广学五题坊（gxwtf）开发规范

广坊旗下站点共用一套技术栈、一套登录、一台主服务器。**先按本规范做，不要自选方案。**

## 开工前

1. `git pull` 再动手——同一个仓库可能有别的会话在改。
2. 确认项目在下面地图里的定位（[references/repos.md](references/repos.md)）。
3. 跑一次现有检查：`pnpm typecheck`，有测试的跑 `pnpm test`。

## 技术栈基线（2026-11）

| 层 | 选型 |
| --- | --- |
| 框架 | Next.js **16.3.5**（App Router）、React **19.3.0** |
| 语言 | TypeScript 6，包管理 **pnpm**（禁止 npm） |
| 样式 | Tailwind CSS 4 + shadcn 风格组件 + lucide-react |
| 数据库 | PostgreSQL（本机 5432），**每站独立库** `gxwtf_<name>` |
| ORM | Prisma **7.10** + `@prisma/adapter-pg` |
| 会话 | iron-session 9 + 广学账号 SSO |
| AI | OpenAI 兼容接口，`LLM_CONFIGS` 多 key 轮换 |
| 构建 | `next build --webpack`（不要 Turbopack） |

细节与版本证据见 [references/conventions.md](references/conventions.md)。

## 认证：三段式，不要发明

```
GET /sso/login?system=<host>&back=<path>
  → 302 https://account.gxwtf.cn/sso/login?system=<host>&back=<path>
  → 回调 <本站>/sso/callback?token=…&back=…
  → 服务端 POST ${GXACCOUNT_URL}/sso/verify {token}
  → 写 iron-session → 302 回 back
```

- cookie 名用**自己的**：`gx<product>_session`，不要蹭别的站；跨子域本来就不共享
- `system` 参数就是请求 Host，**上线前必须让运维在广学账号加白名单**
- 本地联调的 system 是 `localhost:<port>`

## 后端形态

- **业务写入优先 Server Actions**（`"use server"`），首行取会话，**绝不接受前端传 `userId`**
- Route Handler 只留三类：第三方回调（SSO）、流式（SSE）、机器对机器（Bearer）
- 长任务不在请求里同步跑：写任务表 + 后台轮询（见「任务队列」）

## UI 规范

- 颜色只在 `src/app/globals.css` 的 `:root` / `.dark` 成对定义，组件只写语义名
  （`bg-background`、`text-muted-foreground`、`text-brand-red`）
- **禁止** `#hex`、`bg-white`、`text-black`；也不要写 `bg-brand-red/15` 这类任意透明度
  ——Tailwind 可能不生成，改用显式 CSS 类
- 任何前端改动都要**浅色 + 深色**各看一遍
- 有 `pnpm check:colors` 的项目必须跑，0 问题

## 提交与文档

- Conventional Commits + 中文：`feat(收藏): 支持批量导入`、`fix(worker): 转写失败保留原因`
- 一个提交只做一件事；分支持续 `main`
- README 骨架：功能 → 技术栈表 → 快速开始（环境变量表）→ 目录结构 → 数据模型 → 生产部署 → 测试
- 项目文档放 `gxwtf/dev` 仓库的 `<product>/` 目录，并在 `_sidebar.md` 登记；
  服务器上 `cd ~/gxwtf_dev && git pull` 即生效（无 CI）

## 部署（主服务器）

```bash
cd ~/gxwtf_<name> && git pull
pnpm install
pnpm exec prisma migrate deploy
pnpm build
sudo /usr/local/sbin/gxwtf-systemctl restart gx<name>.service
```

- 新服务要先加进 `/usr/local/sbin/gxwtf-systemctl` 的 `ALLOWLIST`
- Nginx 在 `/etc/nginx/sites-enabled/gxwtf` 加 server 块，**必须带**
  `listen 127.0.0.1:8143 ssl proxy_protocol;`（frp 真实 IP），否则公网落到默认站点
- 证书：`HTTPS_PROXY=http://127.0.0.1:7890 certbot --nginx -d <域名>`（直连 Let's Encrypt 会超时）
- 服务器直连 GitHub/PyPI 不通：git 用 SSH remote，pip/curl 走 `127.0.0.1:7890`

## 测试门槛

改完不得直接结束：

1. `pnpm typecheck` 通过
2. `pnpm build` 成功（`.next/BUILD_ID` 生成）
3. 有 `pnpm test` 的跑测试
4. 页面类改动手动过一遍核心路径；有 e2e 的跑 `scripts/e2e/`
5. 部署后 `curl /api/health` + `sh scripts/smoke.sh`

## 任务队列（抓取/转写类项目）

- 站点只写任务表（`FetchJob`），worker 用 `GET /api/worker/jobs` 原子领取
  （`FOR UPDATE SKIP LOCKED`）、`POST /api/worker/jobs/<id>` 回填
- worker **不连数据库**，只走 HTTP + `INTERNAL_API_KEY`
- 失败要写原因，UI 给「重试」，不要静默重试到死

## AI 调用

- 统一走 OpenAI 兼容 `/chat/completions`，不写厂商 SDK
- 配置形态 `LLM_CONFIGS`：JSON 数组，每项一把 key，**按序失败重试**
- 用户自带（BYOK）优先于站点默认；Key 只存库、不回显
- **推理型模型**（deepseek-v4.1-flash 等）要留足 `max_tokens`（≥2000），
  否则 token 全被思考吃光、`content` 为空

## 常见坑速查

| 坑 | 处理 |
| --- | --- |
| `next build` 报 Turbopack | 用 `next build --webpack` |
| Prisma 7 构建时找不到 client | 先 `prisma generate` 再 build |
| `next-mdx-remote` 客户端组件在 RSC 报 `useState of null` | 用 `next-mdx-remote/rsc` |
| Prisma 迁移目录在服务器生成、仓库里没有 | 取回提交，之后统一 `migrate deploy` |
| `pkill -f "pnpm start"` 把自己的 SSH 会话杀了 | 用 `fuser -k <port>/tcp` |
| SSO 回调跳 http | Nginx 没透传 `X-Forwarded-Proto` |
| 服务器 clone 超时 | remote 改 SSH：`git@github.com:gxwtf/<repo>.git` |
| `ctranslate2` / faster-whisper 崩溃 | musl 不支持，改用 whisper.cpp |
| B 站直链不能直接 `<video>` 播 | 是 DASH 分片，需转码或 DASH 播放器 |
| 服务端代理 B 站直链 403 | 必须补 `Referer: https://www.bilibili.com/` + 浏览器 UA |

更多项目地图、端口与 systemd 名见 [references/repos.md](references/repos.md)，
部署排错细节见 [references/deployment.md](references/deployment.md)。
