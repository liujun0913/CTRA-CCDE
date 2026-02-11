# 快速部署指南

## 5分钟快速部署 (Docker方式)

### 前提条件
- 已购买云服务器 (推荐Ubuntu 22.04)
- 已安装Docker和Docker Compose
- 已配置域名DNS解析

### 快速部署步骤

#### 1. 连接服务器
```bash
ssh root@your_server_ip
```

#### 2. 克隆项目
```bash
cd /var/www
git clone https://github.com/your-repo/ctra-ccde.git
cd ctra-ccde
```

#### 3. 配置环境变量
```bash
cp .env.production .env
nano .env
```

编辑以下内容:
```
VITE_API_BASE_URL=https://api.yourdomain.com
VITE_WECHAT_APP_ID=your_wechat_app_id
VITE_WECHAT_APP_SECRET=your_wechat_app_secret
```

#### 4. 构建和启动
```bash
docker-compose up -d
```

#### 5. 配置Nginx反向代理 (可选)
如果需要多域名或SSL配置，参考 [DEPLOYMENT.md](./DEPLOYMENT.md)

#### 6. 访问应用
打开浏览器访问: http://your-domain.com

---

## 传统部署方式 (Nginx)

### 1. 安装Nginx
```bash
apt update
apt install nginx -y
```

### 2. 构建前端
```bash
cd /var/www/ctra-ccde
npm install
npm run build
```

### 3. 配置Nginx
```bash
nano /etc/nginx/sites-available/ctra-ccde
```

添加以下配置:
```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/ctra-ccde/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### 4. 启用站点
```bash
ln -s /etc/nginx/sites-available/ctra-ccde /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

---

## SSL证书配置 (HTTPS)

### 使用Let's Encrypt免费证书
```bash
# 安装Certbot
apt install certbot python3-certbot-nginx -y

# 自动配置SSL
certbot --nginx -d your-domain.com

# 测试自动续期
certbot renew --dry-run
```

---

## 数据库配置

### 安装PostgreSQL
```bash
apt install postgresql postgresql-contrib -y
systemctl start postgresql
systemctl enable postgresql
```

### 创建数据库
```bash
sudo -u postgres psql
```
```sql
CREATE DATABASE ctra_ccde;
CREATE USER ctra_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE ctra_ccde TO ctra_user;
\q
```

---

## 后端API部署 (需要开发)

### 创建后端项目
```bash
mkdir /var/www/backend
cd /var/www/backend
npm init -y
npm install express pg cors dotenv jsonwebtoken
```

### 创建server.js
```javascript
const express = require('express');
const cors = require('cors');
const { Pool } = require('pg');
require('dotenv').config();

const app = express();
const port = 3000;

const pool = new Pool({
  user: process.env.DB_USER,
  host: 'localhost',
  database: process.env.DB_NAME,
  password: process.env.DB_PASSWORD,
  port: 5432,
});

app.use(cors());
app.use(express.json());

app.get('/api/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

### 使用PM2启动
```bash
npm install -g pm2
pm2 start server.js --name backend
pm2 save
pm2 startup
```

---

## 验证部署

### 检查服务状态
```bash
# 检查Nginx
systemctl status nginx

# 检查Docker容器
docker-compose ps

# 检查后端
pm2 status
```

### 测试访问
```bash
# 测试前端
curl http://your-domain.com

# 测试API
curl http://your-domain.com/api/health
```

---

## 常用命令

### Docker命令
```bash
# 查看日志
docker-compose logs -f

# 重启服务
docker-compose restart

# 停止服务
docker-compose down

# 更新部署
git pull
docker-compose build
docker-compose up -d
```

### Nginx命令
```bash
# 测试配置
nginx -t

# 重载配置
nginx -s reload

# 重启服务
systemctl restart nginx
```

### PM2命令
```bash
# 查看日志
pm2 logs

# 重启应用
pm2 restart backend

# 查看状态
pm2 status
```

---

## 故障排查

### 端口被占用
```bash
# 查看端口占用
netstat -tulpn | grep :80

# 停止占用端口的进程
kill -9 <PID>
```

### 权限问题
```bash
# 修改文件权限
chown -R www-data:www-data /var/www/ctra-ccde
chmod -R 755 /var/www/ctra-ccde
```

### 查看错误日志
```bash
# Nginx错误日志
tail -f /var/log/nginx/error.log

# 应用日志
pm2 logs

# Docker日志
docker-compose logs -f
```

---

## 备份和恢复

### 数据库备份
```bash
# 备份
pg_dump -U ctra_user ctra_ccde > backup.sql

# 恢复
psql -U ctra_user ctra_ccde < backup.sql
```

### 自动备份脚本
```bash
# 创建备份目录
mkdir -p /backup

# 创建备份脚本
cat > /usr/local/bin/backup.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
pg_dump -U ctra_user ctra_ccde > /backup/backup_$DATE.sql
find /backup -name "*.sql" -mtime +7 -delete
EOF

chmod +x /usr/local/bin/backup.sh

# 添加定时任务
crontab -e
# 添加: 0 2 * * * /usr/local/bin/backup.sh
```

---

## 联系支持

如有问题，请参考详细文档: [DEPLOYMENT.md](./DEPLOYMENT.md)
