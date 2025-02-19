# 门窗生产管理系统部署文档

## 1. 部署方案概述

### 1.1 部署架构
```mermaid
graph TD
    A[用户] --> B[DNS]
    B --> C[Vercel/负载均衡器]
    C --> D[Next.js应用]
    D --> E[PostgreSQL]
    D --> F[S3存储]
```

### 1.2 部署方案选择

1. **Vercel + AWS方案**
   - 优点：
     * 零运维成本
     * 自动扩展
     * 全球CDN
     * 简单部署
   - 缺点：
     * 成本较高
     * 数据存储受限
     * 定制化受限

2. **自托管方案**
   - 优点：
     * 完全控制
     * 成本可控
     * 灵活定制
   - 缺点：
     * 需要运维团队
     * 配置复杂
     * 扩展性依赖基础设施

## 2. 环境要求

### 2.1 Vercel部署要求

1. **账号准备**
   - Vercel账号
   - AWS账号
   - GitHub账号

2. **资源配置**
   ```env
   # Vercel环境变量
   DATABASE_URL=postgresql://user:pass@host:5432/dbname
   NEXTAUTH_URL=https://your-domain.com
   NEXTAUTH_SECRET=your-secret
   AWS_ACCESS_KEY_ID=your-access-key
   AWS_SECRET_ACCESS_KEY=your-secret-key
   AWS_REGION=your-region
   S3_BUCKET_NAME=your-bucket
   ```

### 2.2 自托管要求

1. **服务器配置**
   ```bash
   # 应用服务器
   CPU: 4核
   内存: 16GB
   磁盘: 100GB SSD
   系统: Ubuntu 22.04 LTS

   # 数据库服务器
   CPU: 4核
   内存: 16GB
   磁盘: 200GB SSD
   系统: Ubuntu 22.04 LTS
   ```

2. **软件要求**
   ```bash
   # 必需软件
   Node.js: 18.x
   PostgreSQL: 16.x
   Docker: 24.x
   Nginx: 1.24.x
   ```

## 3. 部署步骤

### 3.1 Vercel部署

1. **准备工作**
   ```bash
   # 1. Fork项目到自己的GitHub
   # 2. 在Vercel中导入项目
   # 3. 配置环境变量
   ```

2. **数据库配置**
   ```bash
   # 1. 创建AWS RDS实例
   # 2. 配置安全组
   # 3. 运行数据库迁移
   npx prisma migrate deploy
   ```

3. **域名配置**
   ```bash
   # 1. 在Vercel添加域名
   # 2. 配置DNS记录
   # 3. 等待SSL证书生成
   ```

### 3.2 自托管部署

1. **服务器准备**
   ```bash
   # 更新系统
   sudo apt update && sudo apt upgrade -y

   # 安装必要软件
   sudo apt install -y nginx docker.io docker-compose
   ```

2. **Docker配置**
   ```yaml
   # docker-compose.yml
   version: '3.8'
   services:
     app:
       build: .
       ports:
         - "3000:3000"
       environment:
         - DATABASE_URL=postgresql://user:pass@db:5432/dbname
       depends_on:
         - db
     
     db:
       image: postgres:16
       environment:
         - POSTGRES_USER=user
         - POSTGRES_PASSWORD=pass
         - POSTGRES_DB=dbname
       volumes:
         - postgres_data:/var/lib/postgresql/data

   volumes:
     postgres_data:
   ```

3. **Nginx配置**
   ```nginx
   # /etc/nginx/sites-available/default
   server {
       listen 80;
       server_name your-domain.com;

       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

## 4. 配置说明

### 4.1 环境变量
```env
# 应用配置
NODE_ENV=production
PORT=3000

# 数据库配置
DATABASE_URL=postgresql://user:pass@host:5432/dbname

# 认证配置
NEXTAUTH_URL=https://your-domain.com
NEXTAUTH_SECRET=your-secret

# 存储配置
S3_BUCKET_NAME=your-bucket
AWS_REGION=your-region
```

### 4.2 数据库配置
```postgresql
# postgresql.conf
max_connections = 100
shared_buffers = 4GB
effective_cache_size = 12GB
maintenance_work_mem = 1GB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 41943kB
min_wal_size = 1GB
max_wal_size = 4GB
```

## 5. 监控配置

### 5.1 应用监控
```javascript
// 配置Sentry
export const sentryConfig = {
  dsn: "your-sentry-dsn",
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
};
```

### 5.2 服务器监控
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

## 6. 备份策略

### 6.1 数据库备份
```bash
#!/bin/bash
# backup.sh
DATE=$(date +%Y%m%d)
pg_dump -U user dbname > backup_$DATE.sql
aws s3 cp backup_$DATE.sql s3://your-bucket/backups/
```

### 6.2 文件备份
```bash
#!/bin/bash
# files_backup.sh
DATE=$(date +%Y%m%d)
tar -czf uploads_$DATE.tar.gz ./uploads
aws s3 cp uploads_$DATE.tar.gz s3://your-bucket/backups/
```

## 7. 故障恢复

### 7.1 数据库恢复
```bash
# 从备份恢复
psql -U user dbname < backup_20240321.sql

# 检查数据一致性
SELECT count(*) FROM users;
SELECT count(*) FROM orders;
```

### 7.2 应用恢复
```bash
# 回滚部署
git reset --hard HEAD^
git push -f origin main

# 或使用Docker回滚
docker-compose down
docker-compose up -d --build
```

## 8. 维护计划

### 8.1 定期维护
1. **每日维护**
   - 日志检查
   - 备份验证
   - 监控检查

2. **每周维护**
   - 系统更新
   - 性能检查
   - 磁盘清理

### 8.2 更新流程
1. **版本更新**
   ```bash
   # 1. 停止服务
   docker-compose down

   # 2. 更新代码
   git pull origin main

   # 3. 构建新镜像
   docker-compose build

   # 4. 启动服务
   docker-compose up -d
   ```

## 9. 附录

### 9.1 常用命令
```bash
# 查看日志
docker-compose logs -f app

# 数据库备份
pg_dump -U user dbname > backup.sql

# 重启服务
docker-compose restart app
```

### 9.2 故障排查
1. **应用无响应**
   - 检查进程状态
   - 查看错误日志
   - 检查资源使用

2. **数据库问题**
   - 检查连接数
   - 查看慢查询
   - 检查磁盘空间 