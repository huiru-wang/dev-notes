---
title: Next.js TailwindCSS RemoteMDX 博客网站
category: project
tags:
  - project
publishedAt: 2024-06-15
description: 后端程序员初学前端，使用Next.js TailwindCSS RemoteMDX构建的博客网站
---

详见：https://github.com/huiru-wang/blog

# Next.js TailwindCSS RemoteMDX 博客平台

一个后端程序员尝试学习前端，制作的基于 `Next.js`、 `TailwindCSS`、`next-remote-mdx` 的微像素风博客，博客文章是读取本地的md文件。

使用本地文件的方式，主要因为更习惯使用本地的Obsidian写博客文章、开发笔记，单独使用一个仓库。部署时只需要将对应的文章目录迁移到项目的指定读取目录即可；

预览博客：[robinverse.me](https://robinverse.me/)

## 目录
- [简介](#简介)
- [功能特性](#功能特性)
- [安装与配置](#安装与配置)
- [开发](#开发)
- [文件结构](#文件结构)

## 简介

- **Markdown/MDX 支持**：支持 `.md` 和 `.mdx` 文件格式。
- **代码高亮**：使用 `rehype-prism-plus` 插件实现代码块高亮显示，支持单行代码高亮；
- **Markdown目录**：为标题标签（如 `h1`, `h2`）添加 ID，支持目录跳转。
- **多级目录内容支持**：markdown文件从本地读取，默认加载`examples`下的文件，支持多级文件夹结构，自动组装slug，访问对应的blog时解析找到对应文件。
- **Category/Tag筛选**：根据markdown `category` 和 `tag` 进行筛选过滤。
- **自定义Markdown组件**：自定义 markdown渲染组件；
- **黑白主题**：支持黑白主题切换。
- **响应式布局**：响应式布局，支持移动端访问。

![dark](/images/black-home.png)

![white](/images/white-blogs.png)

![markdownwhite](/images/black-markdown.png)

<div style="display: flex;">
<img src="/images/mobile-home.jpg" width="30%" height="30%" center/>
<img src="/images/mobile-blogs.jpg" width="30%" height="30%" center/>
<img src="/images/mobile-projects.jpg" width="30%" height="30%" center/>
</div>


## 安装与配置


```bash
git clone https://github.com/huiru-wang/blog.git
cd blog
```

```bash
pnpm install
```

文件位置：
```env
BLOG_DIR=blogs  # 博客文件存放目录，默认为 "blogs"
```

将md、mdx文件放在blogs目录下即可访问；需配置好frontmatter，否则读取自动跳过
```ts
export type Frontmatter = {
    title: string;
    category: string;
    tags: string[];
    keywords?: string;
    publishedAt?: string;
    description?: string;
}
```

## 启动

启动：

```bash
pnpm dev
```

访问 [http://localhost:3000](http://localhost:3000) 查看博客平台。

## 文件结构

```
blog/
    ├── blogs/
    │   └── 博客文件.md/mdx
    ├── public/
    │   └── ...静态资源
    ├── src/
    │   ├── app/
    │   │   |── blogs/
    │   │   └── projects/
    │   │   └── layout.tsx
    │   │   └── page.tsx
    │   │   
    │   ├── components/
    │   │   └── ...React、Markdown 组件
    │   │  
    │   ├── lib/
    │   │   └── md.ts  # MDX 解析逻辑
    │   │  
    │   ├── styles/
    │   │   └── ...样式文件
    │   │  
    │   ├── providers/
    │   │   └── ThemeProvider.tsx
    │   │  
    ├── .env          # 环境变量配置
    ├── package.json  # 项目依赖
    └── README.md     # 项目说明
```

# 简单部署

## Nginx配置文件

```shell
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        access_log /var/log/nginx/access.log;

        gzip on;

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;

        server {
                listen 80;
                listen [::]:80;
                server_name 106.15.48.213;

                return 301 https://www.robinverse.me$request_uri;
        }

        server {
                set $root /root/blog;

                listen 80;
                listen [::]:80;
                server_name robinverse.me www.robinverse.me;
                return 301 https://$host$request_uri;
        }

        server {
                listen 443 ssl;
                server_name robinverse.me www.robinverse.me;

                ssl_certificate /etc/nginx/ssl/robinverse.me.pem;
                ssl_certificate_key /etc/nginx/ssl/robinverse.me.key;

                location / {
                    proxy_pass http://127.0.0.1:3000;
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection 'upgrade';
                    proxy_set_header Host $host;
                    proxy_cache_bypass $http_upgrade;
                }
            }
    }
```

# 项目启动脚本

项目主要分为2块：
- 博客项目代码；
- 博客文章；（单独仓库）
部署的时候，分别拉取2个仓库的内容，将文章迁移到项目的对应目录，然后启动项目即可；

```shell
#!/bin/bash

TARGET_DIR="/root/blog-website"  # 工作目录
REPO_BLOG_URL="git@github.com:huiru-wang/blog.git"
REPO_BLOG_NAME="blog"
REPO_DEV_NOTE_URL="git@github.com:huiru-wang/dev-notes.git"
REPO_DEV_NOTE_NAME="dev-notes"

SOURCE_DEV_NOTE_DIR="${TARGET_DIR}/${REPO_DEV_NOTE_NAME}"
DEV_NOTE_DIR="${TARGET_DIR}/${REPO_BLOG_NAME}/dev-notes"

SOURCE_PROJECT_DIR="${TARGET_DIR}/${REPO_DEV_NOTE_NAME}/Projects"
PROJECT_DIR="${TARGET_DIR}/${REPO_BLOG_NAME}/blogs"

SOURCE_IMAGES_DIR="${TARGET_DIR}/${REPO_DEV_NOTE_NAME}/images"
DEV_IMAGES_DIR="${TARGET_DIR}/${REPO_BLOG_NAME}/public/images"  # 图片目录

# 1. 切换到工作目录
if [ ! -d "$TARGET_DIR" ]; then
    mkdir -p "$TARGET_DIR"
fi
cd "$TARGET_DIR" || { echo "无法切换到目录 $TARGET_DIR"; exit 1; }

# 2. 拉取项目代码
echo "================= Update Blog ================="
rm -rf "$REPO_BLOG_NAME"
git clone "$REPO_BLOG_URL"

# 3. 拉取文件
echo "================= Update Dev Note ================="
rm -rf "$REPO_DEV_NOTE_NAME"
git clone "$REPO_DEV_NOTE_URL"

# 4. 将文章移动到项目的指定目录
echo "================= Copy Dev Note ================="
if [ -d "$DEV_NOTE_DIR" ]; then
    rm -rf "$DEV_NOTE_DIR"
fi
if [ -d "$PROJECT_DIR" ]; then
    rm -rf "$PROJECT_DIR"
fi
if [ -d "$DEV_IMAGES_DIR" ]; then
    rm -rf "$DEV_IMAGES_DIR"
fi
mkdir -p "$DEV_NOTE_DIR"
mkdir -p "$PROJECT_DIR"
mkdir -p "$DEV_IMAGES_DIR"
mv ${SOURCE_PROJECT_DIR}/* ${PROJECT_DIR}/
mv ${SOURCE_IMAGES_DIR}/* ${DEV_IMAGES_DIR}/
mv ${SOURCE_DEV_NOTE_DIR}/* ${DEV_NOTE_DIR}/
rm -rf "${DEV_NOTE_DIR}/.git"
rm "${DEV_NOTE_DIR}/README.md"

# 5. 构建启动项目
echo "================= Build Blog App ================="
cd "${TARGET_DIR}/${REPO_BLOG_NAME}"
pnpm install
pnpm build

# 6. 重启 pm2 中的应用程序
if pm2 list | grep -q "blog"; then
    # 应用程序已经在 pm2 中，重启它
    echo "===================== Restarting application ====================="
    pm2 restart blog || { echo "Failed to restart pm2 application. Exiting."; exit 1; }
else
    # 应用程序不在 pm2 中，启动它
    echo "===================== Starting application ====================="
    pm2 start pnpm --name 'blog' -- start || { echo "Failed to start pm2 application. Exiting."; exit 1; }
fi
```


# 博客内容热更新脚本

TODO
