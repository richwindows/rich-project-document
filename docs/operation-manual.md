# 门窗生产管理系统运维手册

## 1. 系统架构

### 1.1 系统组件
```mermaid
graph TD
    A[负载均衡器] --> B1[应用服务器1]
    A --> B2[应用服务器2]
    B1 --> C[主数据库]
    B2 --> C
    C --> D[从数据库]
    B1 --> E[文件存储]
    B2 --> E
    F[监控系统] --> B1
    F --> B2
    F --> C
```

### 1.2 技术栈
- Next.js 14
- PostgreSQL 16
- Redis (可选)
- Nginx
- Docker
- Node.js 18

## 2. 日常运维

### 2.1 系统监控

1. **性能监控**
```bash
# 检查系统资源
htop
df -h
free -m

# 检查Docker容器
docker stats
docker ps -a

# 检查数据库
psql -U postgres
SELECT * FROM pg_stat_activity;
```

2. **日志监控**
```bash
# 应用日志
tail -f /var/log/app/error.log
tail -f /var/log/app/access.log

# 数据库日志
tail -f /var/log/postgresql/postgresql-16-main.log

# Nginx日志
tail -f /var/log/nginx/error.log
tail -f /var/log/nginx/access.log
```

### 2.2 定期维护

1. **数据库维护**
```sql
-- 清理过期会话
DELETE FROM sessions WHERE expires < NOW();

-- 更新统计信息
ANALYZE VERBOSE;

-- 清理死锁
SELECT pg_terminate_backend(pid) 
FROM pg_stat_activity 
WHERE state = 'idle in transaction' AND state_change < NOW() - INTERVAL '30 minutes';
```

2. **磁盘维护**
```bash
# 清理日志
find /var/log -name "*.log.*" -mtime +30 -delete

# 清理Docker
docker system prune -af --volumes

# 检查大文件
du -sh /* | sort -hr | head -n 10
```

## 3. 备份恢复

### 3.1 备份策略

1. **数据库备份**
```bash
#!/bin/bash
# /usr/local/bin/backup-db.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/database"

# 全量备份
pg_dump -U postgres dbname > $BACKUP_DIR/full_$DATE.sql

# 压缩备份
gzip $BACKUP_DIR/full_$DATE.sql

# 上传到S3
aws s3 cp $BACKUP_DIR/full_$DATE.sql.gz s3://backup-bucket/database/

# 清理30天前的备份
find $BACKUP_DIR -name "full_*.sql.gz" -mtime +30 -delete
```

2. **文件备份**
```bash
#!/bin/bash
# /usr/local/bin/backup-files.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/files"

# 备份上传文件
tar -czf $BACKUP_DIR/files_$DATE.tar.gz /app/uploads

# 上传到S3
aws s3 cp $BACKUP_DIR/files_$DATE.tar.gz s3://backup-bucket/files/
```

### 3.2 恢复流程

1. **数据库恢复**
```bash
# 从备份恢复
gunzip -c backup.sql.gz | psql -U postgres dbname

# 验证数据
psql -U postgres dbname
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM orders;
```

2. **文件恢复**
```bash
# 恢复文件
tar -xzf files_backup.tar.gz -C /app/uploads

# 修复权限
chown -R www-data:www-data /app/uploads
```

## 4. 性能优化

### 4.1 数据库优化

1. **配置优化**
```ini
# postgresql.conf
max_connections = 200
shared_buffers = 4GB
effective_cache_size = 12GB
maintenance_work_mem = 1GB
work_mem = 41943kB
```

2. **索引优化**
```sql
-- 检查缺失索引
SELECT schemaname, tablename, indexname 
FROM pg_indexes 
ORDER BY tablename, indexname;

-- 检查未使用索引
SELECT * FROM pg_stat_user_indexes;
```

### 4.2 应用优化

1. **Node.js配置**
```bash
# 设置Node环境变量
export NODE_ENV=production
export NODE_OPTIONS="--max-old-space-size=4096"
```

2. **Nginx优化**
```nginx
# nginx.conf
worker_processes auto;
worker_connections 1024;

# 启用gzip
gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

## 5. 安全维护

### 5.1 安全检查

1. **系统安全**
```bash
# 检查系统更新
apt update && apt list --upgradable

# 检查开放端口
netstat -tulpn

# 检查系统日志
journalctl -xe
```

2. **应用安全**
```bash
# 检查依赖漏洞
npm audit

# 检查文件权限
find /app -type f -ls
```

### 5.2 安全加固

1. **防火墙配置**
```bash
# UFW配置
ufw default deny incoming
ufw allow ssh
ufw allow http
ufw allow https
ufw enable
```

2. **SSL配置**
```nginx
# Nginx SSL配置
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers on;
ssl_session_cache shared:SSL:10m;
```

## 6. 故障处理

### 6.1 常见故障

1. **应用故障**
```bash
# 重启应用
docker-compose restart app

# 检查日志
docker-compose logs -f app

# 检查内存
free -m
```

2. **数据库故障**
```bash
# 检查连接数
SELECT count(*) FROM pg_stat_activity;

# 检查锁
SELECT * FROM pg_locks l JOIN pg_stat_activity a ON l.pid = a.pid;

# 检查慢查询
SELECT * FROM pg_stat_activity WHERE state = 'active' AND now() - query_start > '5 minutes'::interval;
```

### 6.2 应急预案

1. **服务降级**
```nginx
# Nginx降级配置
location / {
    return 503;
}
```

2. **数据恢复**
```bash
# 快速恢复步骤
1. 停止应用服务
2. 恢复最近的备份
3. 启动应用服务
4. 验证数据完整性
```

## 7. 监控告警

### 7.1 监控配置

1. **Prometheus配置**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

2. **Grafana配置**
```json
{
  "dashboard": {
    "panels": [
      {
        "title": "CPU Usage",
        "type": "graph"
      },
      {
        "title": "Memory Usage",
        "type": "graph"
      }
    ]
  }
}
```

### 7.2 告警规则

1. **系统告警**
```yaml
# alerts.yml
groups:
  - name: system
    rules:
      - alert: HighCPUUsage
        expr: cpu_usage > 80
        for: 5m
```

2. **应用告警**
```yaml
  - name: application
    rules:
      - alert: HighErrorRate
        expr: error_rate > 5
        for: 5m
```

## 8. 附录

### 8.1 常用命令
```bash
# 系统状态
top
htop
df -h
free -m

# Docker命令
docker ps
docker logs
docker-compose up -d
docker-compose down

# 数据库命令
psql -U postgres
pg_dump
pg_restore
```

### 8.2 配置文件
- `/etc/nginx/nginx.conf`
- `/etc/postgresql/16/main/postgresql.conf`
- `/app/.env`
- `/etc/prometheus/prometheus.yml`

### 8.3 联系信息
- 技术支持：support@example.com
- 运维团队：ops@example.com
- 紧急联系：400-123-4567 