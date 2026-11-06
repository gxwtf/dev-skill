# 项目地图（2026-11）

主服务器：`39.105.54.184:6000`（SSH 非标准端口），内部经 frp 回源。
所有站点由 Nginx 反代，systemd 管理，证书 certbot 自动续签。

## 站点与端口

| 站点 | 域名 | 目录 | 端口 | 服务 |
| --- | --- | --- | --- | --- |
| 主站 | gxwtf.cn | `~/gxwtf_ji` | 3000 | `gxwtf.service` |
| 主站 React | 经主站 | `~/gxwtf_react` | 3002 | `gxreact.service` |
| 广学英语 | english.gxwtf.cn | `~/gxwtf_english` | 3003 | `gxenglish.service` |
| 作文积累本 | writing.gxwtf.cn | `~/gxwtf_writing` | 3004 | `gxwriting.service` |
| 阅读收集 | reading.gxwtf.cn | `~/gxwtf_reading` | 3005 | `gxreading.service` + `gxreading-worker.service` |
| 开发文档 | dev.gxwtf.cn | `~/gxwtf_dev` | 3010 | `gxdev.service` |
| 账号中心 | account.gxwtf.cn | `~/gxaccount` | 3721 | `gxaccount.service` |
| 题库 | ti.gxwtf.cn | `~/gxexercise` | 1229 | `gxexercise.service` |
| 古诗文 | poem.gxwtf.cn | `~/gxwtf_poem` | 1234 | `gxpoem.service` |
| 相册 | photos2.gxwtf.cn | `~/gxphotos` | 3141 | `gxphotos.service` |
| 网盘 | pan.gxwtf.cn | `/opt/openlist` | 5244 | `openlist.service` |
| AI 网关 | ai.gxwtf.cn | `~/new-api` | 8080 | `new-api.service` |

**新增站点前先报备端口与域名**，优先在 3005 之后找空位。

## 仓库一览

| 仓库 | 用途 |
| --- | --- |
| `gxwtf/writing` | 作文积累本（Next 16 + Prisma，规范范本） |
| `gxwtf/english` | 广学英语 |
| `gxwtf/gxexercise` | 题库 |
| `gxwtf/reading-collector` | 阅读收集（Next 站点 + Python worker） |
| `gxwtf/dev` | 开发文档站（Docsify，纯文件即发布） |
| `gxwtf/gxaccount` | 账号中心（Express + MySQL，SSO 提供方） |
| `gxwtf/exam-parser` | 试卷解析流水线（Python） |
| `gxwtf/gxphotos` | 照片直播 + 人脸检索 |
| `gxwtf/docs` | 用户文档站 |

## 账号与身份

- 广学账号：`account.gxwtf.cn`，SSO 唯一来源
- 各站 cookie 名独立：`gxwtf_iron_auth`（english/writing）、`gxexercise_session`、
  `gxreading_session`（阅读收集）
- 跨子域共享登录：目前没有（cookie 未设 domain），不要假设能继承
