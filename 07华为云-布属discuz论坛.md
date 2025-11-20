[toc]

## 在华为云布置discuz论坛



### 一、前期准备

1. **注册并登录华为云**访问[华为云官网](https://www.huaweicloud.com/)，完成注册并登录控制台。

   ![image-20251106112124055](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106112124055.png)

   

### 二、创建网络环境

#### 1. 创建虚拟私有云（VPC）

- 进入华为云控制台，搜索并进入【虚拟私有云】服务。

- 点击【创建虚拟私有云】：

  ![image-20251106112404415](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106112404415.png)

  - 名称：自定义（`discuz-vpc`）
  - 地域：华东 - 上海一
  - 子网：手动设置网段`192.168.0.0/24
  - 其他默认，点击【立即创建】。

#### 2. 创建安全组

- 进入【安全组】页面，点击【创建安全组】：

  ![image-20251106112722359](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106112722359.png)

  ![image-20251106112821338](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106112821338.png)

  - 名称：`discuz-sg`
  - 入方向规则添加：
    - 允许 SSH（22 端口，来源 0.0.0.0/0）
    - 允许 HTTP（80 端口，来源 0.0.0.0/0）
    - 允许 ICMP（ping，可选）
  - 点击【确定】。

### 三、创建 ECS 服务器

#### 1. 购买 ECS 实例

- 进入【弹性云服务器 ECS】控制台，点击【购买弹性云服务器】。

- 基础配置：

  - 计费模式：选择【按需计费】（测试用，用完可释放）
  - 区域：与 VPC 相同地域
  - 规格： 1 核 2G
  - 镜像：选择【公共镜像】→【Ubuntu 24.04 LTS 64 位】

- 网络配置：

  - 虚拟私有云：选择前面创建的 VPC 和子网
  - 安全组：选择`discuz-sg`
  - 弹性公网 IP：勾选【分配弹性公网 IP】

- 登录配置：

  - 认证方式：选择【密码】
    - 用户名：root
    - 密码：discuz123!

- 其他默认，点击【立即购买】并确认。

  ![image-20251106113648928](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106113648928.png)

  ![image-20251106114135823](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106114135823.png)

  创建服务器成功，弹性公网ip为123.60.58.78

  

#### 2. 连接 ECS 服务器

- 打开mac终端，输入ssh root@123.60.58.78
- 提示输入之前配置服务器设置的root密码：discuz123!，ssh 成功
- ![image-20251106114437709](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106114437709.png)

### 四、安装 MySQL 数据库

#### 1. 安装 MySQL

![image-20251106114705553](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106114705553.png)

```bash
sudo apt update  # 更新软件源
sudo apt install -y mysql-server libmysqlclient-dev
```

#### 2. 启动并配置 MySQL

![image-20251106114905507](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106114905507.png)

```bash
# 启动服务并设置开机自启
sudo systemctl start mysql
sudo systemctl enable mysql

# 登录MySQL
sudo mysql -u root

# 设置root密码为Mysql123!
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Mysql23！';
FLUSH PRIVILEGES;

# 创建Discuz专用数据库和用户
CREATE DATABASE discuz CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'discuz_user'@'localhost' IDENTIFIED BY 'DiscuzDB@2025';
GRANT ALL PRIVILEGES ON discuz.* TO 'discuz_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 五、安装 Web 环境（Nginx+PHP）

#### 1. 安装 Nginx

![image-20251106114945835](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106114945835.png)

![image-20251106115016351](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106115016351.png)

```bash
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

访问服务器公网 IP，显示 Nginx 默认页面（说明 Web 服务正常）。

#### 2. 安装 PHP 及扩展

我的ubuntu是22.04

安装系统自带的php8.1		·

![image-20251106120056176](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106120056176.png)

```bash
sudo apt install -y php8.1 php8.1-fpm php8.1-mysql php8.1-gd php8.1-curl php8.1-mbstring php8.1-xml php8.1-zip
```

#### 3. 启动 PHP-FPM

![image-20251106120153872](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106120153872.png)

```bash
systemctl start php8.1-fpm
systemctl enable php8.1-fpm
```

### 六、配置 Nginx 支持 PHP

#### 1. 编辑 Nginx 配置文件



```bash
nano /etc/nginx/sites-available/default
```

#### 2. 修改配置

找到`server { ... }`块，添加 PHP 解析规则和**index.php**：

![image-20251106120323137](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106120323137.png)

```nginx

    # PHP解析规则
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-
```

#### 3. 验证并重启 Nginx

![image-20251106120447614](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106120447614.png)

```bash
nginx -t  # 验证配置（
systemctl restart nginx
```

### 七、部署 Discuz 程序

#### 1. 下载并解压 Discuz



安装解压工具并进入网站根目录，下载使用gitee的链接而后解压文件到根目录并删除原来的程序文件

![image-20251106120748933](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106120748933.png)

```bash
# 安装解压工具
sudo apt install -y unzip

# 进入网站根目录
cd /var/www/html

# 下载最新版Discuz（使用gitee链接）
sudo wget https://gitee.com/Discuz/DiscuzX/releases/download/v3.5-20250901/Discuz_X3.5_TC_UTF8_20250901.zip

# 解压文件
sudo unzip Discuz_X3.5_TC_UTF8_20250901.zip

# 移动程序文件到根目录
sudo mv upload/* .
sudo rm -rf upload Discuz_X3.5_TC_UTF8_20250901.zip
```

#### 2. 设置目录权限

![image-20251106121040041](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106121040041.png)

```bash
# 授予Web服务用户权限
sudo chown -R www-data:www-data /var/www/html

# 配置目录权限
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
sudo chmod -R 775 /var/www/html/data /var/www/html/config /var/www/html/uc_client/data
```

### 八、浏览器完成安装

1. 打开 Mac 浏览器，访问 `http://123.60.58.78/install`
2. 按安装向导操作：
   - 同意协议，选择【全新安装】
   - 数据库配置：
     - 数据库服务器：`localhost`
     - 数据库名：`discuz`
     - 用户名：`discuz_user`
     - 密码：`DiscuzDB@2025`（步骤四中设置的密码）
   - 设置管理员账号密码
   - 点击【下一步】完成安装。

### 九、测试功能

1. 访问 `http://123.60.58.78` 进入论坛首页![image-20251106121535075](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106121535075.png)

   ![image-20251106121954431](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106121954431.png)![image-20251106122007920](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106122007920.png)管理员：admin

   密码：discuz123!![image-20251106122058853](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106122058853.png)

2. 使用管理员账号登录，尝试发表帖子，验证功能正常。

   ![image-20251106122220386](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106122220386.png)

   ![image-20251106122435421](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106122435421.png)

   ![image-20251106122518345](/Users/rileyj_g/Library/Application Support/typora-user-images/image-20251106122518345.png)

成功！