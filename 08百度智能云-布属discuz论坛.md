[toc]





## 在百度云上布属discuz论坛



### 一、前期准备

1. **注册并登录百度智能云账号**访问[百度智能云官网](https://cloud.baidu.com/)，完成注册并登录控制台![image-20251106131225998](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106131225998.png)

### 二、创建网络环境

#### 1. 创建私有网络（VPC）

- 进入百度智能云控制台，搜索并打开【私有网络 VPC】服务。
- 点击【创建私有网络】：
  - 名称：`discuz-vpc`
  - 地域：根据网段自行分配
  - IPv4 网段：`10.0.0.0/16`
  - 点击【确定】。![image-20251106131453155](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106131453155.png)

#### 2. 创建子网

- 在刚创建的 VPC 详情页中，点击【创建子网】：
  - 名称：`discuz-sub1`
  - 可用区：可用区D
  - IPv4 网段：`10.0.2.0/24`
  - 点击【确定】。![image-20251106133356033](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106133356033.png)

#### 3. 创建安全组

- 进入【安全组】服务，点击【创建安全组】：
  - 名称：`discuz-sg`
  - 网络类型：选择`discuz-vpc`
  - 入站规则添加：
    - 协议：SSH，端口：22，源地址：`0.0.0.0/0`（允许 SSH 连接）
    - 协议：HTTP，端口：80，源地址：`0.0.0.0/0`（允许网页访问）
  - 出站规则保持默认（允许所有出站）
  - 点击【确定】。![image-20251106131842400](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106131842400.png)![image-20251106131909333](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106131909333.png)

### 三、创建云服务器

#### 1. 购买 CBB实例

- 进入【弹性云服务器CBB】服务，点击【购买实例】。
- 配置参数：![image-20251106132840338](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106132840338.png)![image-20251106132857058](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106132857058.png)![image-20251106132919044](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106132919044.png)
  - 计费模式：选择 “按量付费”
  - 地域：华北-北京
  - 可用区：随机
  - 实例规格： “计算型” 1核 2G
  - 镜像：选择【公共镜像】→【Ubuntu Server 24.04 LTS 64 位】
  - 系统盘：默认 20GB SSD
  - 网络：选择`discuz-vpc`和`discuz-sub1`
  - 安全组：选择`discuz-sg`
  - 登录方式：密码。root密码:discuz123!
  - 其他默认，点击【立即购买】并确认。

#### 2. 连接 ECS 实例

- 实例创建完成后，在 ECS 列表中找到其【公网 IP】。![image-20251106134141268](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106134141268.png)

- 打开 Mac 终端，ssh连接到服务器

  

  

  

  ![image-20251106134314316](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106134314316.png)

  ```bash
  ssh root@180.76.243.51
  ```

  

### 四、安装 MySQL 数据库

#### 1. 安装 MySQL

![image-20251106134510768](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106134510768.png)

```bash
sudo apt update
sudo apt install -y mysql-server libmysqlclient-dev
```

#### 2. 启动并配置 MySQL



![image-20251106134750196](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106134750196.png)

```bash
# 启动服务并设置开机自启
sudo systemctl start mysql
sudo systemctl enable mysql

# 登录MySQL（Ubuntu默认无密码）
sudo mysql -u root

# 设置root密码（111）
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '111';
FLUSH PRIVILEGES;

# 创建Discuz专用数据库和用户
CREATE DATABASE discuz CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'discuz_user'@'localhost' IDENTIFIED BY 'DiscuzDB@2025';
GRANT ALL PRIVILEGES ON discuz.* TO 'discuz_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 五、安装 Web 环境

#### 1. 安装 Nginx

![image-20251106134903456](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106134903456.png)

```bash
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

访问 EBB公网 IP180.76.243.51，显示 Nginx 默认页面

#### 2. 安装 PHP 及扩展

![image-20251106135128633](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106135128633.png)

```bash
sudo apt install -y php8.1 php8.1-fpm php8.1-mysql php8.1-gd php8.1-curl php8.1-mbstring php8.1-xml php8.1-zip
```

#### 3. 启动 PHP-FPM



```bash
systemctl start php8.1-fpm
systemctl enable php8.1-fpm
```

### 六、配置 Nginx 支持 PHP

#### 1. 编辑 Nginx 配置文件

![image-20251106135226159](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106135226159.png)

```bash
sudo nano /etc/nginx/sites-available/default
```

#### 2. 修改配置

替换`server { ... }`块内容：

![image-20251106140055019](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106140055019.png)



```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    index index.php index.html index.htm;
    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }
}
```

按`Ctrl+O`保存，`Ctrl+X`退出。

#### 3. 验证并重启 Nginx

![image-20251106135402779](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106135402779.png)

```bash
sudo nginx -t  
sudo systemctl restart nginx
```

### 七、部署 Discuz 程序

#### 1. 下载并解压 Discuz

![image-20251106135502392](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106135502392.png)

```bash
sudo apt install -y unzip  # 安装解压工具
cd /var/www/html  # 进入网站根目录

# 下载最新版Discuz（使用gitee链接）
sudo wget https://gitee.com/Discuz/DiscuzX/releases/download/v3.5-20250901/Discuz_X3.5_TC_UTF8_20250901.zip

# 解压并移动文件
sudo unzip Discuz_X3.5_TC_UTF8_20250901.zip
sudo mv upload/* .
sudo rm -rf upload Discuz_X3.5_TC_UTF8_20250901.zip
```

#### 2. 设置目录权限

确保discuz安装的时候有权限

```bash
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
sudo chmod -R 775 /var/www/html/data /var/www/html/config /var/www/html/uc_client/data
```

### 八、浏览器完成安装

1. 打开 Mac 浏览器，访问 `http://180.76.243.51/install`

   ![image-20251106140141280](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106140141280.png)

1. 按安装向导操作：
   - 同意协议，选择【全新安装】。
   - 数据库配置：
   - ![image-20251106140301423](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106140301423.png)![image-20251106140319289](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106140319289.png)
     - 数据库服务器：`localhost`
     - 数据库名：`discuz`
     - 用户名：`discuz_user`
     - 密码：`DiscuzDB@2025`
   - 设置管理员账号密码，完成安装。

### 九、测试功能

1. 访问 `http://你的ECS公网IP` 进入论坛首页。

   ![image-20251106140406331](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106140406331.png)

   ![image-20251106140652377](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106140652377.png)![image-20251106140745310](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106140745310.png)

2. 登录管理员账号，尝试发表帖子，验证功能正常。