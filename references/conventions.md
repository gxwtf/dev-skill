# 约定细节

## 版本证据（取自 2026-10/11 的 package.json）

```json
{
  "next": "16.3.5",
  "react": "19.3.0",
  "react-dom": "19.3.0",
  "typescript": "^6",
  "@prisma/client": "^7.10.0",
  "@prisma/adapter-pg": "^7.10.0",
  "iron-session": "^9.0.1",
  "tailwindcss": "^4",
  "radix-ui": "^1.6.7",
  "lucide-react": "^1.45.0",
  "next-themes": "^0.4.6",
  "sonner": "^2.0.8"
}
```

脚本：`dev` / `build: next build --webpack` / `start` / `typecheck: tsc --noEmit` /
`test: node --test src/lib`（纯函数测试）/ `db:*`（prisma）。

## Prisma 7 写法

```ts
// src/lib/db.ts
import { PrismaClient } from "../generated/prisma/client";
import { PrismaPg } from "@prisma/adapter-pg";
import pg from "pg";

const pool = new pg.Pool({ connectionString: process.env.DATABASE_URL });
export const prisma = globalThis.__prisma ?? new PrismaClient({ adapter: new PrismaPg(pool) });
```

- generator 用 `provider = "prisma-client"`，输出 `../src/generated/prisma`
- `prisma.config.ts` 里指定 schema 与 migrations 路径
- 生产只跑 `migrate deploy`；`migrate dev` 只允许 localhost
- JSON 字段写入用 `Prisma.InputJsonValue` / `Prisma.JsonNull`，不要 `as never`

## 数据建模惯例

- `User` 是 SSO 镜像：`userId @unique`（广学账号 ID），业务表外键都指向它
- 长 URL 不要直接建唯一索引：存 `urlHash`（sha256）再 `@@unique([userId, urlHash])`
- 列表类表统一带 `@@index([userId, createdAt])`
- 状态用字符串列（`pending/ready/failed`）而不是枚举，方便跨版本演进

## Server Action 模板

```ts
"use server";
import { revalidatePath } from "next/cache";
import { prisma } from "@/lib/db";
import { requireSession } from "@/lib/session";

export async function doSomething(id: number) {
  const session = await requireSession("/");
  const row = await prisma.entry.findFirst({
    where: { id, userId: session.userId },   // 永远带 userId
    select: { id: true },
  });
  if (!row) return;                           // 别人的数据当不存在
  await prisma.entry.update({ where: { id }, data: { /* … */ } });
  revalidatePath("/");
}
```

## 文件与导出

- Route Handler 只能导出 HTTP 方法与 `dynamic` 等配置，**多导出一个工具函数会导致构建失败**
- `"use server"` 文件只能导出 async 函数，工具函数不要 export
- 需要复用逻辑就抽到 `src/lib/`，Action 与 Route 共用
