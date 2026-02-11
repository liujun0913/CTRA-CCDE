# 中国陶行知研究会儿童发展与教育专委会管理系统 - 部署指南

## 目录
1. [系统架构](#系统架构)
2. [服务器要求](#服务器要求)
3. [云服务商选择](#云服务商选择)
4. [部署方案](#部署方案)
5. [详细部署步骤](#详细部署步骤)
6. [域名和SSL配置](#域名和ssl配置)
7. [监控和维护](#监控和维护)

---

## 系统架构

### 前端架构
- **技术栈**: React 18 + TypeScript + Vite
- **UI框架**: Ant Design 5
- **路由**: React Router 6
- **部署方式**: 静态文件部署 (Nginx)

### 后端架构 (需要开发)
- **技术栈**: Node.js / Python / Java (推荐)
- **API框架**: Express / FastAPI / Spring Boot
- **认证**: JWT + 微信小程序认证
- **数据库**: PostgreSQL / MySQL

### 数据存储
- **数据库**: PostgreSQL 14+ 或 MySQL 8.0+
- **缓存**: Redis 6.0+ (可选，用于会话管理)
- **文件存储**: 对象存储服务 (阿里云OSS / 腾讯云COS)

---

## 服务器要求

### 操作系统
推荐使用以下操作系统之一：

#### 1. Linux (推荐)
- **Ubuntu 20.04 LTS / 22.04 LTS** (推荐)
- **CentOS 7 / 8**
- **Debian 10 / 11**

#### 2. Windows Server
- **Windows Server 2019 / 2022**
- 需要额外配置 IIS 或使用 Docker

### 硬件配置

#### 最小配置 (测试/开发环境)
- CPU: 1核
- 内存: 1GB
- 存储: 20GB SSD
- 带宽: 1Mbps

#### 推荐配置 (生产环境)
- CPU: 2核或以上
- 内存: 4GB或以上
- 存储: 40GB SSD或以上
- 带宽: 5Mbps或以上

#### 高可用配置 (大型部署)
- CPU: 4核或以上
- 内存: 8GB或以上
- 存储: 100GB SSD或以上
- 带宽: 10Mbps或以上

---

## 云服务商选择

### 国内云服务商

#### 1. 阿里云
- **ECS云服务器**: https://www.aliyun.com/product/ecs
- **RDS数据库**: https://www.aliyun.com/product/rds
- **OSS对象存储**: https://www.aliyun.com/product/oss
- **SLB负载均衡**: https://www.aliyun.com/product/slb

#### 2. 腾讯云
- **CVM云服务器**: https://cloud.tencent.com/product/cvm
- **MySQL数据库**: https://cloud.tencent.com/product/cdb
- **COS对象存储**: https://cloud.tencent.com/product/cos
- **CLB负载均衡**: https://cloud.tencent.com/product/clb

#### 3. 华为云
- **ECS云服务器**: https://www.huaweicloud.com/product/ecs.html
- **RDS数据库**: https://www.huaweicloud.com/product/rds.html
- **OBS对象存储**: https://www.huaweicloud.com/product/obs.html

### 国际云服务商

#### 1. AWS
- **EC2**: https://aws.amazon.com/ec2/
- **RDS**: https://aws.amazon.com/rds/
- **S3**: https://aws.amazon.com/s3/

#### 2. Google Cloud
- **Compute Engine**: https://cloud.google.com/compute
- **Cloud SQL**: https://cloud.google.com/sql
- **Cloud Storage**: https://cloud.google.com/storage

---

## 部署方案

### 方案一: 传统部署 (适合小型项目)

#### 架构图
```
用户 → Nginx → 静态文件 (React应用)
           ↓
        后端API (Node.js/Python/Java)
           ↓
        数据库 (PostgreSQL/MySQL)
```

#### 优点
- 成本低
- 配置简单
- 适合快速上线

#### 缺点
- 扩展性有限
- 单点故障风险

### 方案二: Docker容器化部署 (推荐)

#### 架构图
```
用户 → Nginx (反向代理)
     ↓
  Docker容器
     ├─ Frontend (React)
     ├─ Backend (API)
     └─ Database (PostgreSQL)
```

#### 优点
- 环境一致性
- 易于扩展
- 快速部署

#### 缺点
- 需要Docker知识
- 资源占用稍高

### 方案三: Kubernetes部署 (适合大型项目)

#### 架构图
```
用户 → 负载均衡
     ↓
  Kubernetes集群
     ├─ Frontend Pods
     ├─ Backend Pods
     └─ Database StatefulSet
```

#### 优点
- 高可用
- 自动扩展
- 自愈能力

#### 缺点
- 配置复杂
- 成本较高
- 需要专业知识

---

## 详细部署步骤

### 方案一: 传统部署 (Ubuntu + Nginx)

#### 1. 购买服务器
以阿里云为例:
1. 登录阿里云控制台
2. 选择"云服务器ECS"
3. 点击"创建实例"
4. 配置参数:
   - 实例规格: 2核4GB
   - 镜像: Ubuntu 22.04 LTS
   - 存储: 40GB SSD
   - 网络: 默认专有网络
5. 设置密码或SSH密钥
6. 确认订单并支付

#### 2. 连接服务器
```bash
# 使用SSH连接
ssh root@your_server_ip

# 或使用密钥
ssh -i your_key.pem root@your_server_ip
```

#### 3. 安装必要软件
```bash
# 更新系统
apt update && apt upgrade -y

# 安装Nginx
apt install nginx -y

# 安装Node.js (用于构建)
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs

# 安装Git
apt install git -y

# 安装PM2 (进程管理)
npm install -g pm2
```

#### 4. 安装和配置数据库
```bash
# 安装PostgreSQL
apt install postgresql postgresql-contrib -y

# 启动PostgreSQL服务
systemctl start postgresql
systemctl enable postgresql

# 登录PostgreSQL
sudo -u postgres psql

# 创建数据库和用户
CREATE DATABASE ctra_ccde;
CREATE USER ctra_user WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE ctra_ccde TO ctra_user;
\q
```

#### 5. 部署前端应用
```bash
# 克隆代码
cd /var/www
git clone https://github.com/your-repo/ctra-ccde.git
cd ctra-ccde

# 安装依赖
npm install

# 构建生产版本
npm run build

# 配置Nginx
nano /etc/nginx/sites-available/ctra-ccde
```

Nginx配置文件内容:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    root /var/www/ctra-ccde/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

```bash
# 启用站点
ln -s /etc/nginx/sites-available/ctra-ccde /etc/nginx/sites-enabled/

# 测试配置
nginx -t

# 重启Nginx
systemctl restart nginx
```

#### 6. 部署后端API (需要开发)
```bash
# 创建后端目录
mkdir -p /var/www/backend
cd /var/www/backend

# 初始化Node.js项目
npm init -y

# 安装依赖
npm install express pg cors dotenv jsonwebtoken bcrypt

# 创建server.js (示例)
cat > server.js << 'EOF'
const express = require('express');
const cors = require('cors');
const { Pool } = require('pg');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 3000;

const pool = new Pool({
  user: process.env.DB_USER,
  host: process.env.DB_HOST,
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
EOF

# 创建环境变量文件
cat > .env << 'EOF'
PORT=3000
DB_USER=ctra_user
DB_HOST=localhost
DB_NAME=ctra_ccde
DB_PASSWORD=your_secure_password
JWT_SECRET=your_jwt_secret
EOF

# 使用PM2启动
pm2 start server.js --name backend
pm2 save
pm2 startup
```

#### 7. 配置防火墙
```bash
# 允许HTTP和HTTPS
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 22/tcp
ufw enable
```

---

### 方案二: Docker部署 (推荐)

#### 1. 安装Docker
```bash
# 安装Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# 安装Docker Compose
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# 验证安装
docker --version
docker-compose --version
```

#### 2. 准备项目文件
项目根目录已包含:
- `Dockerfile` - 前端容器配置
- `nginx.conf` - Nginx配置
- `docker-compose.yml` - Docker编排配置

#### 3. 构建和运行
```bash
# 克隆代码
cd /var/www
git clone https://github.com/your-repo/ctra-ccde.git
cd ctra-ccde

# 构建并启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 查看运行状态
docker-compose ps
```

#### 4. 更新部署
```bash
# 拉取最新代码
git pull

# 重新构建
docker-compose build

# 重启服务
docker-compose up -d
```

---

## 域名和SSL配置

### 1. 购买域名
推荐域名服务商:
- 阿里云: https://wanwang.aliyun.com/
- 腾讯云: https://dnspod.cloud.tencent.com/
- Namecheap: https://www.namecheap.com/

### 2. 配置DNS解析
在域名服务商控制台添加A记录:
```
类型: A
主机记录: @
记录值: your_server_ip
TTL: 600
```

### 3. 申请SSL证书
推荐使用Let's Encrypt免费证书:

#### 安装Certbot
```bash
# 安装Certbot
apt install certbot python3-certbot-nginx -y

# 自动配置SSL
certbot --nginx -d your-domain.com

# 自动续期
certbot renew --dry-run
```

#### 手动配置Nginx SSL
```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    root /var/www/ctra-ccde/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}
```

---

## 监控和维护

### 1. 系统监控
推荐工具:
- **Prometheus + Grafana**: 完整的监控解决方案
- **Zabbix**: 企业级监控系统
- **Uptime Robot**: 简单的网站监控

### 2. 日志管理
```bash
# 查看Nginx日志
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# 查看应用日志 (PM2)
pm2 logs

# 查看Docker日志
docker-compose logs -f
```

### 3. 备份策略
```bash
# 数据库备份脚本
cat > /usr/local/bin/backup-db.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup"
pg_dump -U ctra_user ctra_ccde > $BACKUP_DIR/ctra_ccde_$DATE.sql
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
EOF

chmod +x /usr/local/bin/backup-db.sh

# 添加到crontab (每天凌晨2点备份)
crontab -e
# 添加: 0 2 * * * /usr/local/bin/backup-db.sh
```

### 4. 安全加固
```bash
# 禁用root远程登录
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart sshd

# 配置防火墙
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable

# 安装fail2ban防止暴力破解
apt install fail2ban -y
systemctl enable fail2ban
systemctl start fail2ban
```

---

## 成本估算

### 阿里云 (月度成本)
- ECS服务器 (2核4GB): ¥200-300
- RDS数据库 (基础版): ¥150-200
- OSS存储 (10GB): ¥1-2
- 带宽 (5Mbps): ¥100-150
- **总计**: ¥450-650/月

### 腾讯云 (月度成本)
- CVM服务器 (2核4GB): ¥200-300
- MySQL数据库 (基础版): ¥150-200
- COS存储 (10GB): ¥1-2
- 带宽 (5Mbps): ¥100-150
- **总计**: ¥450-650/月

---

## 常见问题

### Q1: 如何选择云服务商?
A: 根据您的需求选择:
- 国内用户首选阿里云或腾讯云
- 需要国际化选择AWS或Google Cloud
- 考虑成本可以选择华为云或百度云

### Q2: 数据库选择PostgreSQL还是MySQL?
A: 
- PostgreSQL: 功能强大，适合复杂查询
- MySQL: 简单易用，社区活跃
- 本项目推荐PostgreSQL

### Q3: 需要开发后端吗?
A: 是的，当前只有前端界面，需要开发后端API来处理:
- 用户认证
- 数据存储
- 业务逻辑
- 微信小程序接口

### Q4: 如何实现微信小程序登录?
A: 需要:
1. 注册微信小程序账号
2. 获取AppID和AppSecret
3. 开发后端接口处理微信登录
4. 实现实名认证流程

### Q5: 部署后如何更新?
A: 
- 传统部署: git pull + npm run build + 重启服务
- Docker部署: git pull + docker-compose build + docker-compose up -d
- 推荐使用CI/CD自动化部署

---

## 技术支持

如有问题，请联系:
- 技术支持邮箱: support@example.com
- 项目文档: https://docs.example.com
- 问题反馈: https://github.com/your-repo/issues

---

**最后更新**: 2026-02-11
**版本**: 1.0.0
