# 后端API开发指南

## 技术栈推荐

### 方案一: Node.js + Express (推荐)
- 优点: 与前端技术栈一致，开发效率高
- 适合: 中小型项目，快速开发

### 方案二: Python + FastAPI
- 优点: 性能好，代码简洁，自动生成文档
- 适合: 需要高性能的项目

### 方案三: Java + Spring Boot
- 优点: 企业级框架，稳定性高
- 适合: 大型企业项目

---

## 项目结构

```
backend/
├── src/
│   ├── config/          # 配置文件
│   │   ├── database.js
│   │   └── wechat.js
│   ├── controllers/     # 控制器
│   │   ├── auth.controller.js
│   │   ├── member.controller.js
│   │   └── project.controller.js
│   ├── models/         # 数据模型
│   │   ├── user.model.js
│   │   ├── member.model.js
│   │   └── project.model.js
│   ├── routes/         # 路由
│   │   ├── auth.routes.js
│   │   ├── member.routes.js
│   │   └── project.routes.js
│   ├── services/       # 业务逻辑
│   │   ├── auth.service.js
│   │   ├── wechat.service.js
│   │   └── email.service.js
│   ├── middleware/     # 中间件
│   │   ├── auth.middleware.js
│   │   └── error.middleware.js
│   ├── utils/          # 工具函数
│   │   ├── jwt.util.js
│   │   └── validator.util.js
│   └── app.js         # 应用入口
├── .env               # 环境变量
├── package.json
└── README.md
```

---

## 核心API接口

### 1. 认证接口

#### 微信登录
```
POST /api/auth/wechat/login
Request:
{
  "code": "wx_login_code"
}

Response:
{
  "success": true,
  "token": "jwt_token",
  "userInfo": {
    "id": "user_id",
    "openid": "wechat_openid",
    "nickName": "用户昵称",
    "avatarUrl": "头像URL",
    "isRealNameAuth": false
  }
}
```

#### 实名认证
```
POST /api/auth/real-name
Headers:
  Authorization: Bearer {token}

Request:
{
  "realName": "张三",
  "idCard": "身份证号",
  "phone": "手机号"
}

Response:
{
  "success": true,
  "message": "实名认证成功"
}
```

#### 传统登录
```
POST /api/auth/login
Request:
{
  "username": "admin",
  "password": "password"
}

Response:
{
  "success": true,
  "token": "jwt_token",
  "userInfo": {...}
}
```

#### 退出登录
```
POST /api/auth/logout
Headers:
  Authorization: Bearer {token}

Response:
{
  "success": true,
  "message": "退出成功"
}
```

### 2. 会员管理接口

#### 会员注册
```
POST /api/members/register
Request:
{
  "memberType": "personal",
  "name": "张三",
  "phone": "13800138000",
  "email": "zhangsan@example.com",
  "company": "北京大学",
  ...
}

Response:
{
  "success": true,
  "message": "注册成功，等待审核"
}
```

#### 会员查询
```
GET /api/members/query?name=张三&memberType=personal&page=1&limit=10
Headers:
  Authorization: Bearer {token}

Response:
{
  "success": true,
  "data": [...],
  "total": 100,
  "page": 1,
  "limit": 10
}
```

#### 会员审核
```
POST /api/members/:id/approve
Headers:
  Authorization: Bearer {token}

Request:
{
  "status": "approved",
  "comment": "审核通过"
}

Response:
{
  "success": true,
  "message": "审核成功"
}
```

### 3. 课题管理接口

#### 课题申报
```
POST /api/projects/apply
Headers:
  Authorization: Bearer {token}

Request:
{
  "projectName": "课题名称",
  "projectType": "theory",
  "applicant": "张三",
  "phone": "13800138000",
  "email": "zhangsan@example.com",
  "company": "北京大学",
  "projectSummary": "课题摘要",
  "researchContent": "研究内容",
  "expectedResults": "预期成果",
  "projectFund": 50000
}

Response:
{
  "success": true,
  "message": "申报成功"
}
```

#### 课题查询
```
GET /api/projects?page=1&limit=10&status=pending
Headers:
  Authorization: Bearer {token}

Response:
{
  "success": true,
  "data": [...],
  "total": 50
}
```

#### 课题评估
```
POST /api/projects/:id/evaluate
Headers:
  Authorization: Bearer {token}

Request:
{
  "score": 90,
  "comment": "优秀",
  "status": "approved"
}

Response:
{
  "success": true,
  "message": "评估成功"
}
```

---

## 数据库设计

### 用户表 (users)
```sql
CREATE TABLE users (
  id VARCHAR(36) PRIMARY KEY,
  openid VARCHAR(100) UNIQUE NOT NULL,
  username VARCHAR(50),
  password VARCHAR(255),
  nick_name VARCHAR(100),
  avatar_url VARCHAR(500),
  real_name VARCHAR(50),
  id_card VARCHAR(20),
  phone VARCHAR(20),
  is_real_name_auth BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 会员表 (members)
```sql
CREATE TABLE members (
  id VARCHAR(36) PRIMARY KEY,
  user_id VARCHAR(36) REFERENCES users(id),
  member_type VARCHAR(20) NOT NULL,
  name VARCHAR(100) NOT NULL,
  gender VARCHAR(10),
  phone VARCHAR(20) NOT NULL,
  email VARCHAR(100),
  company VARCHAR(200),
  address TEXT,
  unit_name VARCHAR(200),
  unit_level VARCHAR(20),
  legal_representative VARCHAR(50),
  contact_person VARCHAR(50),
  contact_phone VARCHAR(20),
  contact_email VARCHAR(100),
  unit_address TEXT,
  unit_intro TEXT,
  payment_method VARCHAR(20),
  payment_proof VARCHAR(500),
  status VARCHAR(20) DEFAULT 'pending',
  join_date DATE,
  expire_date DATE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 课题表 (projects)
```sql
CREATE TABLE projects (
  id VARCHAR(36) PRIMARY KEY,
  user_id VARCHAR(36) REFERENCES users(id),
  project_name VARCHAR(200) NOT NULL,
  project_type VARCHAR(20) NOT NULL,
  applicant VARCHAR(50) NOT NULL,
  phone VARCHAR(20) NOT NULL,
  email VARCHAR(100) NOT NULL,
  company VARCHAR(200) NOT NULL,
  start_date DATE,
  end_date DATE,
  project_summary TEXT,
  research_content TEXT,
  expected_results TEXT,
  project_fund DECIMAL(10,2),
  application_file VARCHAR(500),
  status VARCHAR(20) DEFAULT 'pending',
  score INTEGER DEFAULT 0,
  comment TEXT,
  progress VARCHAR(20) DEFAULT '0%',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 微信小程序配置

### 1. 注册小程序
1. 访问 https://mp.weixin.qq.com/
2. 注册小程序账号
3. 完成认证（需要企业资质）
4. 获取 AppID 和 AppSecret

### 2. 配置服务器域名
在小程序后台配置:
- request合法域名: https://api.yourdomain.com
- uploadFile合法域名: https://api.yourdomain.com
- downloadFile合法域名: https://api.yourdomain.com

### 3. 实现微信登录流程
```javascript
// 前端调用
wx.login({
  success: (res) => {
    // 发送code到后端
    fetch('/api/auth/wechat/login', {
      method: 'POST',
      body: JSON.stringify({ code: res.code })
    })
  }
})

// 后端处理
async function wechatLogin(code) {
  // 1. 通过code换取openid和session_key
  const response = await axios.get(
    `https://api.weixin.qq.com/sns/jscode2session?appid=${APPID}&secret=${SECRET}&js_code=${code}&grant_type=authorization_code`
  )
  
  const { openid, session_key } = response.data
  
  // 2. 查询或创建用户
  let user = await User.findOne({ openid })
  if (!user) {
    user = await User.create({ openid })
  }
  
  // 3. 生成JWT token
  const token = jwt.sign({ userId: user.id }, JWT_SECRET)
  
  return { token, userInfo: user }
}
```

---

## 环境变量配置

```env
# 服务器配置
PORT=3000
NODE_ENV=production

# 数据库配置
DB_HOST=localhost
DB_PORT=5432
DB_NAME=ctra_ccde
DB_USER=ctra_user
DB_PASSWORD=your_password

# JWT配置
JWT_SECRET=your_jwt_secret_key
JWT_EXPIRES_IN=7d

# 微信小程序配置
WECHAT_APP_ID=your_app_id
WECHAT_APP_SECRET=your_app_secret

# 文件上传配置
UPLOAD_DIR=/uploads
MAX_FILE_SIZE=10485760

# 邮件配置
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=noreply@example.com
SMTP_PASS=your_password
```

---

## 安全建议

### 1. 认证安全
- 使用JWT进行身份认证
- Token设置合理的过期时间
- 敏感操作需要二次验证

### 2. 数据安全
- 密码使用bcrypt加密
- 身份证号加密存储
- 使用HTTPS传输

### 3. API安全
- 实现请求频率限制
- 验证所有输入数据
- 使用CORS限制跨域访问

### 4. 日志记录
- 记录所有API请求
- 记录用户操作日志
- 定期备份日志

---

## 部署建议

### 1. 使用PM2管理进程
```bash
npm install -g pm2
pm2 start src/app.js --name backend
pm2 save
pm2 startup
```

### 2. 配置Nginx反向代理
```nginx
location /api {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
}
```

### 3. 使用环境变量
```bash
# 生产环境使用.env.production
NODE_ENV=production node src/app.js
```

---

## 测试

### 单元测试
```bash
npm install --save-dev jest
npm test
```

### API测试
推荐工具:
- Postman
- Insomnia
- curl

### 压力测试
```bash
npm install -g autocannon
autocannon -c 10 -d 30 http://localhost:3000/api/health
```

---

## 监控和日志

### 1. 日志管理
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});
```

### 2. 性能监控
- 使用APM工具: New Relic, Datadog
- 或自建监控: Prometheus + Grafana

---

## 联系支持

如有问题，请参考:
- 前端文档: [SYSTEM_DOCUMENTATION.md](./SYSTEM_DOCUMENTATION.md)
- 部署文档: [DEPLOYMENT.md](./DEPLOYMENT.md)
