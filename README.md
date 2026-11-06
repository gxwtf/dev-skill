# gxwtf-dev

广学五题坊（gxwtf）的开发规范与项目地图，打包成一个 **agent skill**（技能）。
给 AI agent 用，也给新加入的同学当速查手册。

- 组织：<https://github.com/gxwtf>
- 作者：广学五题坊（gxwtf）· [@kyoukawa](https://github.com/kyoukawa)
- 线上项目示例：<https://reading.gxwtf.cn> · 文档站 <https://dev.gxwtf.cn>

## 里面有什么

| 文件 | 内容 |
| --- | --- |
| `SKILL.md` | 主入口：开工流程、技术栈基线、SSO 契约、后端/UI/提交规范、部署与测试门槛、常见坑 |
| `references/repos.md` | 站点地图：域名、目录、端口、systemd 服务名 |
| `references/conventions.md` | 版本证据、Prisma 7 写法、数据建模惯例、Server Action 模板 |
| `references/deployment.md` | 发布流水、Nginx/systemd 模板、出网受限处理、排错顺序 |

## 怎么用

**给 agent 用**：把整个目录放进技能目录即可。

```bash
git clone https://github.com/gxwtf/dev-skill.git
cp -r dev-skill ~/.claude/skills/gxwtf-dev
# Minis：cp -r dev-skill /var/minis/skills/gxwtf-dev
```

之后 agent 遇到「在 gxwtf 仓库改代码 / 建新站 / 接 SSO / 部署」会自动加载。

**给人看**：直接读 `SKILL.md`，10 分钟能摸清这个组织怎么干活。

## 维护

- 规范变了就改对应文件，并在提交信息里写清楚依据（版本号、线上验证结果）
- 新增站点：补 `references/repos.md` 的表格和端口占用
- 踩到新坑：写进 `SKILL.md` 的「常见坑速查」，一行一条，别写故事

## 许可

MIT，见 [LICENSE](LICENSE)。

---

Built and maintained by 广学五题坊（gxwtf）with [@kyoukawa](https://github.com/kyoukawa).
