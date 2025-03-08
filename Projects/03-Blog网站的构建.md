
![](/images/blog-home-page.png)

详见：https://github.com/huiru-wang/blog


# Nginx配置

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
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

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
- 博客文章；

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


