# Vercel 可直接部署的完整项目源码

项目名称：student-supervision-app

技术栈：

* Next.js 15
* React
* TailwindCSS
* Prisma
* MySQL
* JWT

---

# 一、项目目录结构

```text
student-supervision-app
├── app
│   ├── api
│   │   ├── login
│   │   │   └── route.js
│   │   ├── register
│   │   │   └── route.js
│   │   └── tasks
│   │       └── route.js
│   ├── dashboard
│   │   └── page.jsx
│   ├── login
│   │   └── page.jsx
│   ├── globals.css
│   ├── layout.js
│   └── page.jsx
│
├── lib
│   ├── auth.js
│   └── prisma.js
│
├── prisma
│   └── schema.prisma
│
├── .env
├── next.config.js
├── package.json
├── postcss.config.js
├── tailwind.config.js
└── jsconfig.json
```

---

# 二、package.json

```json
{
  "name": "student-supervision-app",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "@prisma/client": "latest",
    "axios": "latest",
    "bcryptjs": "latest",
    "framer-motion": "latest",
    "jsonwebtoken": "latest",
    "mysql2": "latest",
    "next": "latest",
    "react": "latest",
    "react-dom": "latest",
    "react-icons": "latest",
    "recharts": "latest"
  },
  "devDependencies": {
    "autoprefixer": "latest",
    "postcss": "latest",
    "prisma": "latest",
    "tailwindcss": "latest"
  }
}
```

---

# 三、next.config.js

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
}

module.exports = nextConfig
```

---

# 四、tailwind.config.js

```javascript
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

---

# 五、postcss.config.js

```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

---

# 六、jsconfig.json

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

---

# 七、.env

```env
DATABASE_URL="mysql://root:password@localhost:3306/student_app"
JWT_SECRET="super_secret_key"
```

---

# 八、Prisma 数据库模型

## prisma/schema.prisma

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  password  String
  role      String
  createdAt DateTime @default(now())

  tasks Task[]
}

model Task {
  id         Int      @id @default(autoincrement())
  title      String
  progress   Int
  focusScore Int
  online     Boolean
  userId     Int
  createdAt  DateTime @default(now())

  user User @relation(fields: [userId], references: [id])
}
```

---

# 九、Prisma 客户端

## lib/prisma.js

```javascript
import { PrismaClient } from '@prisma/client'

const globalForPrisma = global

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient()

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}
```

---

# 十、JWT 工具

## lib/auth.js

```javascript
import jwt from 'jsonwebtoken'

export function generateToken(user) {
  return jwt.sign(user, process.env.JWT_SECRET, {
    expiresIn: '7d',
  })
}
```

---

# 十一、全局样式

## app/globals.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  margin: 0;
  padding: 0;
  background: #f1f5f9;
  font-family: sans-serif;
}
```

---

# 十二、根布局

## app/layout.js

```javascript
import './globals.css'

export const metadata = {
  title: '学生监督学习平台',
}

export default function RootLayout({ children }) {
  return (
    <html lang="zh-CN">
      <body>{children}</body>
    </html>
  )
}
```

---

# 十三、首页

## app/page.jsx

```jsx
import Link from 'next/link'

export default function Home() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-slate-100">
      <div className="bg-white p-10 rounded-3xl shadow-xl text-center max-w-2xl">
        <h1 className="text-5xl font-black text-slate-800">
          AI 学生监督学习平台
        </h1>

        <p className="text-slate-500 mt-6 text-lg leading-relaxed">
          面向补习班、自习室、晚辅导与线上教育的智能监督学习系统。
        </p>

        <div className="flex justify-center gap-4 mt-8">
          <Link
            href="/login"
            className="bg-blue-600 text-white px-8 py-4 rounded-2xl font-bold"
          >
            登录系统
          </Link>

          <Link
            href="/dashboard"
            className="bg-slate-800 text-white px-8 py-4 rounded-2xl font-bold"
          >
            查看后台
          </Link>
        </div>
      </div>
    </div>
  )
}
```

---

# 十四、登录页面

## app/login/page.jsx

```jsx
'use client'

import { useState } from 'react'
import axios from 'axios'
import { useRouter } from 'next/navigation'

export default function LoginPage() {
  const router = useRouter()

  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')

  async function handleLogin() {
    const res = await axios.post('/api/login', {
      email,
      password,
    })

    localStorage.setItem('token', res.data.token)

    router.push('/dashboard')
  }

  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="bg-white p-10 rounded-3xl shadow-xl w-full max-w-md">
        <h1 className="text-3xl font-bold text-center mb-8">
          平台登录
        </h1>

        <div className="space-y-4">
          <input
            type="email"
            placeholder="邮箱"
            className="w-full border p-4 rounded-xl"
            onChange={(e) => setEmail(e.target.value)}
          />

          <input
            type="password"
            placeholder="密码"
            className="w-full border p-4 rounded-xl"
            onChange={(e) => setPassword(e.target.value)}
          />

          <button
            onClick={handleLogin}
            className="w-full bg-blue-600 text-white py-4 rounded-xl font-bold"
          >
            登录
          </button>
        </div>
      </div>
    </div>
  )
}
```

---

# 十五、教师后台

## app/dashboard/page.jsx

```jsx
export default function Dashboard() {
  const students = [
    {
      name: '张明',
      progress: 88,
      focus: 95,
    },
    {
      name: '李华',
      progress: 72,
      focus: 84,
    },
  ]

  return (
    <div className="min-h-screen p-8 bg-slate-100">
      <div className="max-w-7xl mx-auto">
        <div className="bg-white rounded-3xl p-8 shadow-lg mb-8">
          <h1 className="text-4xl font-black">
            教师监督后台
          </h1>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          {students.map((student, index) => (
            <div
              key={index}
              className="bg-white rounded-3xl p-6 shadow-lg"
            >
              <h2 className="text-2xl font-bold">
                {student.name}
              </h2>

              <div className="mt-4">
                学习进度：{student.progress}%
              </div>

              <div className="mt-2">
                AI专注度：{student.focus}%
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  )
}
```

---

# 十六、登录 API

## app/api/login/route.js

```javascript
import { prisma } from '@/lib/prisma'
import bcrypt from 'bcryptjs'
import { generateToken } from '@/lib/auth'

export async function POST(req) {
  const body = await req.json()

  const user = await prisma.user.findUnique({
    where: {
      email: body.email,
    },
  })

  if (!user) {
    return Response.json({ error: '用户不存在' })
  }

  const valid = await bcrypt.compare(
    body.password,
    user.password
  )

  if (!valid) {
    return Response.json({ error: '密码错误' })
  }

  const token = generateToken(user)

  return Response.json({ token, user })
}
```

---

# 十七、注册 API

## app/api/register/route.js

```javascript
import { prisma } from '@/lib/prisma'
import bcrypt from 'bcryptjs'

export async function POST(req) {
  const body = await req.json()

  const hashedPassword = await bcrypt.hash(body.password, 10)

  const user = await prisma.user.create({
    data: {
      name: body.name,
      email: body.email,
      password: hashedPassword,
      role: body.role,
    },
  })

  return Response.json(user)
}
```

---

# 十八、Tasks API

## app/api/tasks/route.js

```javascript
import { prisma } from '@/lib/prisma'

export async function GET() {
  const tasks = await prisma.task.findMany({
    include: {
      user: true,
    },
  })

  return Response.json(tasks)
}
```

---

# 十九、本地运行

```bash
npm install

npx prisma migrate dev --name init

npm run dev
```

打开：

```text
http://localhost:3000
```

---

# 二十、上传 GitHub

```bash
git init
git add .
git commit -m "student app"
git branch -M main
git remote add origin 你的仓库地址
git push -u origin main
```

---

# 二十一、Vercel 部署

打开：

[https://vercel.com](https://vercel.com)

导入 GitHub 项目。

部署成功后：

```text
https://你的项目名.vercel.app
```

即可真正在线访问。

