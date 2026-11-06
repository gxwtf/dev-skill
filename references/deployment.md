# 部署与排错

## 发布流水

```bash
cd ~/gxwtf_<name>
git pull                     # 若 https 不通，先改 SSH remote
pnpm install                 # 依赖变化时
pnpm exec prisma migrate deploy
pnpm build                   # 必须成功，确认 .next/BUILD_ID
sudo /usr/local/sbin/gxwtf-systemctl restart gx<name>.service
curl -s localhost:<port>/api/health
```

> 只 restart 不 build 等于没改：`next start` 服务的是 `.next` 里的旧产物。

## Nginx 新站模板

```nginx
server {
    server_name foo.gxwtf.cn;
    location / {
        proxy_pass http://127.0.0.1:3xxx;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;   # 写死，别用 $proxy_add_x_forwarded_for
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    listen 80;
}

# certbot 加完 443 后，同一个 server 块补一行（frp 真实 IP，必做）
listen 127.0.0.1:8143 ssl proxy_protocol;
```

漏了 8143 那行的表现：公网打开显示别的站点 / 444 / 404。

## systemd 模板

```systemd
[Unit]
Description=gxwtf <name> Server
After=postgresql.service

[Service]
User=gxwtf
Group=gxwtf
WorkingDirectory=/home/gxwtf/gxwtf_<name>/
EnvironmentFile=/home/gxwtf/gxwtf_<name>/.env
ExecStart=/usr/local/bin/pnpm start
Restart=always

[Install]
WantedBy=multi-user.target
```

注意 `pnpm` 用绝对路径 `/usr/local/bin/pnpm`，否则 `status=203/EXEC`。

## 出网受限

- GitHub：只有 SSH key 能出网，`git remote set-url origin git@github.com:gxwtf/<repo>.git`
- PyPI / npm / certbot：`HTTPS_PROXY=http://127.0.0.1:7890`
- 部分时段 GitHub 直接拒推（`commit_refs_rules`），可用 `git bundle` + scp 中转部署

## 排错顺序

1. `systemctl status gx<name>` / `journalctl -u gx<name> -n 50`
2. `curl -s localhost:<port>/api/health`
3. `nginx -t`、`ss -lntp | grep <port>`
4. 构建产物：`ls .next/BUILD_ID`
5. 数据库：`pg_isready`、`psql -c 'select 1'`

## 备份

- 数据库：`pg_dump -U gxwtf -Fc gxwtf_<name> > /var/backups/<date>.dump`
- 媒体/附件目录单独备份
- `.env` 与 `prisma/migrations/` 必须保留
