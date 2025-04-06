# Magento 2 部署前检查清单详细说明

## 前言
本文档提供了 Magento 2 部署前的完整检查清单，包含每个配置项的详细解释和具体实现方法。建议在部署前逐项检查确认。

## 1. 系统环境要求

### 1.1 先决条件检查命令
以下是验证基础环境的关键命令：

```bash
# Apache 版本检查
## CentOS 系统使用：
httpd -v
## Ubuntu 系统使用：
apache2 -v
# 预期输出应显示 2.4.x 版本

# PHP 版本检查
php -v
# 预期输出应显示 7.4 或更高版本

# MySQL 版本检查
mysql -V
# 预期输出应显示 8.0 或更高版本

# 搜索引擎检查
## OpenSearch 检查
curl -XGET '<opensearch-hostname>:<opensearch-port>'
## Elasticsearch 检查
curl -XGET '<elasticsearch-hostname>:<elasticsearch-port>'
# 预期输出应显示版本信息和服务状态
```

### 1.2 Apache 要求详解
Apache 2.4 是必需的，因为它提供：
- 更好的性能优化选项
- 更强的安全性功能
- 更灵活的配置选项

必需的 Apache 模块：
- mod_rewrite：用于 URL 重写，实现友好链接
- mod_ssl：提供 SSL/TLS 支持
- mod_deflate：提供内容压缩功能

检查模块是否启用：
```bash
apache2ctl -M | grep -E 'rewrite|ssl|deflate'
```

### 1.3 PHP 配置详解

#### 1.3.1 版本要求
- Magento 2.4.x 需要 PHP 7.4 或更高版本
- 推荐使用 PHP 8.1 以获得最佳性能
- 不同 Magento 版本对应的 PHP 版本要求：
  - Magento 2.4.0-2.4.3：PHP 7.4
  - Magento 2.4.4+：PHP 8.1
  - Magento 2.4.6+：PHP 8.2

#### 1.3.2 PHP 扩展详细说明
必需的 PHP 扩展及其用途：

```bash
# 检查所有必需的 PHP 扩展
php -m | grep -E "bcmath|ctype|curl|dom|gd|hash|iconv|intl|mbstring|openssl|pdo_mysql|simplexml|soap|xsl|zip|sodium"
```

扩展详解：
1. bcmath
   - 用途：处理高精度数学计算
   - 应用：商品价格计算、货币转换
   - 重要性：关键，影响交易准确性

2. ctype
   - 用途：字符类型检查
   - 应用：表单验证、数据过滤
   - 重要性：基础安全组件

3. curl
   - 用途：处理 HTTP 请求
   - 应用：API 集成、支付网关对接
   - 重要性：核心功能组件

4. dom
   - 用途：XML 文档处理
   - 应用：配置文件处理、数据导入导出
   - 重要性：系统基础组件

5. gd
   - 用途：图片处理
   - 应用：产品图片缩放、水印添加
   - 重要性：媒体处理必需

6. hash
   - 用途：数据加密和校验
   - 应用：密码加密、数据完整性验证
   - 重要性：安全核心组件

7. iconv
   - 用途：字符集转换
   - 应用：多语言支持、编码转换
   - 重要性：国际化必需组件

8. intl
   - 用途：国际化支持
   - 应用：货币格式化、日期本地化
   - 重要性：多语言商店必需

9. mbstring
   - 用途：多字节字符处理
   - 应用：UTF-8字符处理、字符串操作
   - 重要性：多语言支持核心

10. openssl
    - 用途：加密和解密功能
    - 应用：安全通信、密码加密
    - 重要性：安全传输必需

11. pdo_mysql
    - 用途：MySQL 数据库连接
    - 应用：数据库操作
    - 重要性：数据存储核心

12. simplexml
    - 用途：XML 处理
    - 应用：配置文件、数据交换
    - 重要性：系统配置必需

13. soap
    - 用途：Web 服务支持
    - 应用：第三方系统集成
    - 重要性：API 集成必需

14. xsl
    - 用途：XML 转换
    - 应用：数据转换、报表生成
    - 重要性：数据处理必需

15. zip
    - 用途：压缩文件处理
    - 应用：备份、导入导出
    - 重要性：数据传输必需

16. sodium
    - 用途：现代加密库
    - 应用：高级加密功能
    - 重要性：安全增强组件

## 2. 服务配置检查

### MySQL 配置
- 版本要求：8.0+
- 检查版本：`mysql --version`
- 关键配置参数：
  ```ini
  innodb_buffer_pool_size = 1G (最小)
  max_allowed_packet = 64M
  wait_timeout = 28800
  ```

### Redis 配置
- 检查运行状态：`redis-cli ping`
- 建议配置：
  ```conf
  maxmemory 2gb
  maxmemory-policy allkeys-lru
  ```
- 分别配置三个实例：
  - 默认缓存（端口 6379，数据库 0）
  - 页面缓存（端口 6379，数据库 1）
  - 会话存储（端口 6379，数据库 2）

### Elasticsearch/OpenSearch
- 版本要求：7.x 或更高
- 检查运行状态：`curl -X GET 'http://localhost:9200'`
- 内存配置：最少 2GB

### 2.3 Elasticsearch/OpenSearch 详细配置

#### 基础配置
```yaml
# 集群配置
cluster.name: magento2
node.name: magento2_node1

# 网络设置
network.host: 0.0.0.0
http.port: 9200

# 内存设置
-Xms2g
-Xmx2g

# 路径配置
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

#### 重要参数说明
1. 内存配置
   - Xms：初始堆内存（建议与 Xmx 相同）
   - Xmx：最大堆内存（建议为系统内存的 50%）

2. 索引设置
```yaml
index.number_of_shards: 1
index.number_of_replicas: 0  # 开发环境
index.number_of_replicas: 1  # 生产环境
```

### 2.4 Web 服务器配置详解

#### Apache 虚拟主机完整配置
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName magento.local
    ServerAlias www.magento.local
    DocumentRoot /var/www/html/magento2ce/pub
    
    # 日志配置
    ErrorLog ${APACHE_LOG_DIR}/magento_error.log
    CustomLog ${APACHE_LOG_DIR}/magento_access.log combined
    
    # SSL 重定向（如果需要）
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
    
    # 目录配置
    <Directory "/var/www/html/magento2ce/pub">
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
        
        # 性能优化
        EnableMMAP Off
        EnableSendfile On
        
        # PHP 配置（如果使用 mod_php）
        php_value memory_limit 2G
        php_value max_execution_time 1800
    </Directory>
    
    # 压缩配置
    AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css text/javascript application/javascript
    
    # 缓存控制
    <FilesMatch "\.(ico|pdf|flv|jpg|jpeg|png|gif|js|css|swf)$">
        Header set Cache-Control "max-age=31536000, public"
    </FilesMatch>
</VirtualHost>
```

#### Nginx 配置示例（可选）
```nginx
upstream fastcgi_backend {
    server unix:/run/php/php8.1-fpm.sock;
}

server {
    listen 80;
    server_name magento.local;
    set $MAGE_ROOT /var/www/html/magento2ce;
    set $MAGE_MODE developer; # 或 production
    
    # SSL 配置（如果需要）
    listen 443 ssl;
    ssl_certificate /etc/ssl/certs/magento.crt;
    ssl_certificate_key /etc/ssl/private/magento.key;
    
    include /var/www/html/magento2ce/nginx.conf.sample;
}
```

## 3. 文件系统权限

### 关键目录权限设置
```bash
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
```

### 目录说明
- var：临时文件和缓存
- pub/static：静态文件
- pub/media：上传的媒体文件
- app/etc：配置文件

### 3.1 权限设置原则
1. 安全性考虑
   - 文件权限不应过于开放
   - 避免使用 777 权限
   - 遵循最小权限原则

2. 用户组设置
```bash
# 创建专用用户组
groupadd magento

# 添加 Web 服务器用户到组
usermod -a -G magento www-data

# 设置目录所有者
chown -R www-data:magento /var/www/html/magento2ce
```

3. 特殊目录权限说明
```bash
# 配置文件目录
chmod 750 app/etc
chmod 640 app/etc/*.xml

# 媒体文件目录
chmod 750 pub/media
chmod -R g+w pub/media

# 静态文件目录
chmod 750 pub/static
chmod -R g+w pub/static

# var 目录
chmod 750 var
chmod -R g+w var

# 生成文件目录
chmod 750 generated
chmod -R g+w generated
```

### 3.2 权限检查工具
```bash
# 创建权限检查脚本
cat > magento-permissions-check.sh << 'EOF'
#!/bin/bash
echo "检查 Magento 2 关键目录权限..."

MAGENTO_ROOT="/var/www/html/magento2ce"
DIRS_TO_CHECK="app/etc var pub/static pub/media generated"

for dir in $DIRS_TO_CHECK; do
    echo "检查 $dir:"
    find "$MAGENTO_ROOT/$dir" -type f -exec stat -c "%a %n" {} \;
done
EOF

chmod +x magento-permissions-check.sh
```

## 4. 内存限制详细配置

### 4.1 PHP 内存配置详解
```ini
# 基本内存限制
memory_limit = 2G                # 推荐值，可根据需求调整
max_execution_time = 1800        # 30分钟执行时限
max_input_time = 1800           # 数据接收时限
post_max_size = 64M             # POST 数据大小限制
upload_max_filesize = 64M       # 上传文件大小限制
max_input_vars = 10000          # 最大输入变量数

# 会话配置
session.gc_maxlifetime = 1440   # 会话最大生命周期
session.save_handler = redis    # 使用 Redis 存储会话
```

### 4.2 MySQL 内存配置详解
```ini
# InnoDB 缓冲池配置
innodb_buffer_pool_size = 1G    # 最小值，建议为系统内存的 50-75%
innodb_buffer_pool_instances = 4 # 缓冲池实例数，建议为 CPU 核心数

# 查询缓存配置
query_cache_size = 64M          # 查询缓存大小
query_cache_type = 1            # 启用查询缓存

# 临时表配置
tmp_table_size = 64M            # 内存临时表大小
max_heap_table_size = 64M       # 堆表大小

# 连接配置
max_connections = 1000          # 最大连接数
thread_cache_size = 8           # 线程缓存大小
```

### 4.3 Elasticsearch 内存配置详解
```yaml
# JVM 堆配置
-Xms2g                          # 初始堆大小
-Xmx2g                          # 最大堆大小

# GC 配置
-XX:+UseG1GC                    # 使用 G1 垃圾收集器
-XX:G1HeapRegionSize=32M       # G1 区域大小

# 内存锁定
bootstrap.memory_lock: true     # 防止内存交换

# 缓存配置
indices.memory.index_buffer_size: 10%  # 索引缓冲区大小
indices.queries.cache.size: 5%         # 查询缓存大小
```

## 5. 域名和 SSL 详细配置

### 5.1 本地开发环境配置
```bash
# hosts 文件位置
## Windows: C:\Windows\System32\drivers\etc\hosts
## Linux/Mac: /etc/hosts

# 添加本地域名
127.0.0.1 magento.local
127.0.0.1 www.magento.local
```

### 5.2 SSL 证书配置
#### 开发环境（自签名证书）
```bash
# 生成自签名证书
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/magento.key \
    -out /etc/ssl/certs/magento.crt \
    -subj "/C=US/ST=State/L=City/O=Organization/CN=magento.local"

# Apache 配置
SSLEngine on
SSLCertificateFile /etc/ssl/certs/magento.crt
SSLCertificateKeyFile /etc/ssl/private/magento.key
```

#### 生产环境（Let's Encrypt）
```bash
# 安装 Certbot
apt-get install certbot python3-certbot-apache

# 获取证书
certbot --apache -d example.com -d www.example.com

# 自动续期配置
0 0 1 * * /usr/bin/certbot renew --quiet
```

## 6. Composer 相关

### Composer 版本
- 检查版本：`composer --version`
- 推荐使用 Composer 2.x

### auth.json 配置
```json
{
    "http-basic": {
        "repo.magento.com": {
            "username": "YOUR_MAGENTO_PUBLIC_KEY",
            "password": "YOUR_MAGENTO_PRIVATE_KEY"
        }
    }
}
```

## 7. 数据库相关

### 创建数据库
```sql
CREATE DATABASE magento2 CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

### 用户权限
```sql
GRANT ALL ON magento2.* TO 'magento'@'localhost';
```

## 8. 缓存配置

### Redis 内存配置
- 总内存至少 2GB
- 持久化配置：
```conf
save 900 1
save 300 10
save 60 10000
```

## 9. 定时任务

### Cron 配置
```bash
*/5 * * * * /usr/bin/php /var/www/magento2/bin/magento cron:run
```

### 检查 Cron 状态
```bash
bin/magento cron:status
```

## 10. 性能优化相关

### PHP OPcache 配置
```ini
opcache.enable = 1
opcache.memory_consumption = 256
opcache.max_accelerated_files = 60000
```

### PHP-FPM 配置
```ini
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
```

## 11. 开发模式设置

### 设置运行模式
```bash
bin/magento deploy:mode:set developer  # 开发环境
bin/magento deploy:mode:set production # 生产环境
```

### Xdebug 配置（仅开发环境）
```ini
xdebug.mode = debug
xdebug.client_host = localhost
xdebug.client_port = 9003
```

## 12. 安全相关

### 文件权限
- 文件权限：644
- 目录权限：755
- 特殊目录：775

### 管理员路径
```bash
bin/magento admin:url:set /custom_admin
```

## 13. 备份策略

### 数据库备份
```bash
bin/magento setup:backup --db
```

### 代码备份
```bash
bin/magento setup:backup --code
```

## 14. 部署后检查项

### 部署静态文件
```bash
bin/magento setup:static-content:deploy -f
```

### 清理缓存
```bash
bin/magento cache:clean
bin/magento cache:flush
```

### 检查项目
1. 访问前台确认：
   - 页面加载速度
   - 图片显示
   - 搜索功能
   - 产品列表
   
2. 访问后台确认：
   - 登录功能
   - 订单管理
   - 产品管理
   - 缓存管理

3. 功能测试：
   - 产品搜索
   - 购物车
   - 结账流程
   - 支付功能

## 15. 日志管理

### 日志轮换配置
```bash
# 创建 logrotate 配置
vim /etc/logrotate.d/magento2
```

配置示例：
```conf
/var/www/html/magento2/var/log/*.log {
    weekly
    rotate 4
    missingok
    notifempty
    compress
    dateext
    create 640 www-data www-data
}
```

## 16. 服务器安全设置

### SELinux 配置（如果启用）
```bash
# 允许 Apache 访问文件系统
setsebool -P httpd_can_network_connect 1
setsebool -P httpd_can_network_connect_db 1
```

### iptables 配置
```bash
# Web 服务器端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# MySQL 端口（如果使用远程数据库）
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT

# Elasticsearch/OpenSearch 端口
iptables -A INPUT -p tcp --dport 9200 -j ACCEPT

# Redis 端口
iptables -A INPUT -p tcp --dport 6379 -j ACCEPT
```

## 17. 电子邮件服务器设置

### Postfix 配置（推荐）
```bash
# 安装 Postfix
apt-get install postfix # Ubuntu
yum install postfix # CentOS

# 配置 Postfix
vim /etc/postfix/main.cf
```

基本配置示例：
```conf
myhostname = magento.local
mydomain = magento.local
myorigin = $mydomain
inet_interfaces = all
inet_protocols = ipv4
```

## 18. 消息队列配置

### RabbitMQ 设置（Adobe Commerce 2.3.0+）
```bash
# 安装 RabbitMQ
apt-get install rabbitmq-server # Ubuntu
yum install rabbitmq-server # CentOS

# 创建用户和虚拟主机
rabbitmqctl add_user magento magento
rabbitmqctl add_vhost magento
rabbitmqctl set_permissions -p magento magento ".*" ".*" ".*"
```

## 19. 部署流程检查清单

1. 备份
```bash
# 数据库备份
bin/magento setup:backup --db
# 代码备份
bin/magento setup:backup --code
```

2. 维护模式
```bash
# 启用维护模式
bin/magento maintenance:enable
```

3. 代码部署
```bash
# 拉取代码
git pull origin master

# 更新依赖
composer install --no-dev

# 更新数据库架构
bin/magento setup:upgrade

# 编译代码
bin/magento setup:di:compile

# 部署静态内容
bin/magento setup:static-content:deploy -f

# 清理缓存
bin/magento cache:clean
bin/magento cache:flush
```

4. 权限修复
```bash
# 设置权限
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
```

5. 禁用维护模式
```bash
bin/magento maintenance:disable
```

## 注意事项
- 部署前备份所有数据
- 先在开发环境测试
- 记录所有配置变更
- 准备回滚方案
- 确保所有服务（MySQL, Redis, Elasticsearch/OpenSearch, RabbitMQ）都已正确配置并运行
- 检查日志轮换配置是否正确
- 验证电子邮件服务器配置
- 确保防火墙规则正确配置
- 记录所有配置变更以便回滚

## 20. 性能优化指南

### 20.1 服务器级优化

#### Nginx FastCGI 缓存配置
```nginx
# /etc/nginx/conf.d/magento_fastcgi_cache.conf
fastcgi_cache_path /tmp/nginx-cache levels=1:2 keys_zone=MAGENTO:100m inactive=60m;
fastcgi_cache_key "$request_method|$http_host|$request_uri|$cookie_PHPSESSID";
fastcgi_cache_use_stale error timeout invalid_header http_500;
fastcgi_cache_valid 200 60m;
```

#### PHP-FPM 高级配置
```ini
; /etc/php/8.1/fpm/pool.d/magento.conf
[magento]
user = www-data
group = www-data
listen = /run/php/php8.1-fpm-magento.sock
listen.owner = www-data
listen.group = www-data
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500
request_terminate_timeout = 600s
```

#### MySQL 查询优化
```sql
-- 添加常用查询的索引
ALTER TABLE catalog_product_entity_varchar ADD INDEX attribute_id_value (attribute_id, value);
ALTER TABLE catalog_category_product ADD INDEX category_id_product_id (category_id, product_id);
```

### 20.2 应用级优化

#### 启用生产模式优化
```bash
# 启用生产模式
bin/magento deploy:mode:set production

# 启用所有优化
bin/magento setup:performance:generate-fixtures profile.xml
```

#### Redis 会话优化
```php
// app/etc/env.php
'session' => [
    'save' => 'redis',
    'redis' => [
        'host' => '127.0.0.1',
        'port' => '6379',
        'database' => '2',
        'compression_threshold' => '2048',
        'compression_library' => 'gzip',
        'max_concurrency' => '6',
        'break_after_frontend' => '5',
        'break_after_adminhtml' => '30',
        'first_lifetime' => '600',
        'bot_first_lifetime' => '60',
        'bot_lifetime' => '7200',
        'disable_locking' => '0',
        'min_lifetime' => '60',
        'max_lifetime' => '2592000'
    ]
]
```

## 21. 故障排除指南

### 21.1 常见问题检查清单

1. 页面加载缓慢
```bash
# 检查 PHP-FPM 状态
systemctl status php8.1-fpm

# 检查 MySQL 慢查询
tail -f /var/log/mysql/mysql-slow.log

# 检查 Redis 连接
redis-cli ping
redis-cli info | grep connected_clients
```

2. 缓存问题
```bash
# 清理所有缓存
bin/magento cache:flush

# 检查 Redis 内存使用
redis-cli info | grep used_memory_human

# 重建索引
bin/magento indexer:reindex
```

3. 权限问题
```bash
# 修复权限脚本
#!/bin/bash
cd /var/www/html/magento2
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chmod u+x bin/magento
```

### 21.2 日志分析工具

```bash
# 创建日志分析脚本
cat > magento-log-analyzer.sh << 'EOF'
#!/bin/bash
LOG_DIR="/var/www/html/magento2/var/log"

echo "=== 错误日志分析 ==="
grep -i "error" $LOG_DIR/system.log | tail -n 20

echo "=== 异常日志分析 ==="
grep -i "exception" $LOG_DIR/exception.log | tail -n 20

echo "=== 调试日志分析 ==="
grep -i "debug" $LOG_DIR/debug.log | tail -n 20
EOF

chmod +x magento-log-analyzer.sh
```

## 22. 监控配置

### 22.1 系统监控

```bash
# 创建系统资源监控脚本
cat > magento-monitor.sh << 'EOF'
#!/bin/bash
echo "=== 系统资源使用情况 ==="
echo "CPU 使用率:"
top -bn1 | grep "Cpu(s)" | awk '{print $2}'

echo "内存使用情况:"
free -m

echo "磁盘使用情况:"
df -h

echo "=== PHP-FPM 进程 ==="
ps aux | grep php-fpm | grep -v grep

echo "=== MySQL 连接数 ==="
mysql -e "SHOW STATUS WHERE Variable_name = 'Threads_connected';"

echo "=== Redis 状态 ==="
redis-cli info | grep connected_clients
EOF

chmod +x magento-monitor.sh
```

### 22.2 应用监控

```php
// 添加到 app/bootstrap.php
if (PHP_SAPI !== 'cli') {
    $startTime = microtime(true);
    register_shutdown_function(function () use ($startTime) {
        $endTime = microtime(true);
        $executionTime = $endTime - $startTime;
        
        if ($executionTime > 2) { // 超过2秒的请求
            file_put_contents(
                'var/log/slow_requests.log',
                date('Y-m-d H:i:s') . " - " . 
                $_SERVER['REQUEST_URI'] . " - " .
                round($executionTime, 2) . "s\n",
                FILE_APPEND
            );
        }
    });
}
```

## 最终检查清单

### 部署前检查
- [ ] 所有系统要求已满足
- [ ] 所有必需的服务已配置
- [ ] 数据库已备份
- [ ] 文件权限已正确设置
- [ ] SSL 证书已配置
- [ ] Cron 作业已设置
- [ ] 缓存服务已配置

### 部署后检查
- [ ] 前台页面可访问
- [ ] 后台可登录
- [ ] 产品搜索功能正常
- [ ] 图片显示正常
- [ ] 缓存工作正常
- [ ] 邮件系统工作正常
- [ ] 支付系统工作正常

### 性能检查
- [ ] 页面加载时间 < 2秒
- [ ] 数据库查询优化
- [ ] 缓存命中率 > 80%
- [ ] 静态文件已压缩
- [ ] CDN 配置正确

### 安全检查
- [ ] 所有密码都已更改
- [ ] 管理员URL已自定义
- [ ] 防火墙规则已配置
- [ ] SSL/TLS 配置安全
- [ ] 文件权限正确

## 结束语
本部署指南涵盖了 Magento 2 部署的主要方面。请根据具体环境和需求调整配置参数。建议在进行生产环境部署之前，先在测试环境中完整执行一遍部署流程。
