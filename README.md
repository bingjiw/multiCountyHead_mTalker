# 多县长mTalker 文档



##服务器设置多用户 vsftpd 结构。以下是完整配置步骤：

### 1. 基础配置

```bash
# 安装 vsftpd
sudo apt update
sudo apt install vsftpd

# 备份原配置
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

### 2. 创建基础目录结构
```bash
# 创建FTP根目录
sudo mkdir -p /home/ftp_root

# 设置权限
sudo chmod 755 /home/ftp_root
```

### 3. 创建用户和目录的脚本
```bash
#!/bin/bash
# 保存为 create_ftp_user.sh

create_ftp_user() {
    USERNAME=$1
    PASSWORD=$2

    # 创建用户
    sudo useradd -m -d "/home/ftp_root/FTP_$USERNAME" -s /sbin/nologin $USERNAME
    echo "$USERNAME:$PASSWORD" | sudo chpasswd

    # 创建必要的目录
    sudo mkdir -p "/home/ftp_root/FTP_$USERNAME/Dir-ItchatMsg"
    sudo mkdir -p "/home/ftp_root/FTP_$USERNAME/Dir-SendingOut"
    sudo mkdir -p "/home/ftp_root/FTP_$USERNAME/tmp"

    # 设置所有权
    sudo chown -R $USERNAME:$USERNAME "/home/ftp_root/FTP_$USERNAME"

    # 设置权限
    sudo chmod 755 "/home/ftp_root/FTP_$USERNAME"
    sudo chmod 755 "/home/ftp_root/FTP_$USERNAME/Dir-ItchatMsg"
    sudo chmod 755 "/home/ftp_root/FTP_$USERNAME/Dir-SendingOut"
    sudo chmod 755 "/home/ftp_root/FTP_$USERNAME/tmp"

    echo "用户 $USERNAME 创建完成"
}

# 使用示例
create_ftp_user "UserAAA" "password123"
create_ftp_user "UserBBB" "password456"
create_ftp_user "UserCCC" "password789"
```

### 4. vsftpd 配置文件
```conf
# /etc/vsftpd.conf

# 基本设置
listen=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES

# 用户限制设置
chroot_local_user=YES
allow_writeable_chroot=YES
user_sub_token=$USER
local_root=/home/ftp_root/FTP_$USER

# 被动模式设置
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=50000

# 日志设置
xferlog_file=/var/log/vsftpd.log
xferlog_std_format=YES
log_ftp_protocol=YES
```

### 5. 启动服务
```bash
# 重启 vsftpd 服务
sudo systemctl restart vsftpd

# 检查状态
sudo systemctl status vsftpd
```

### 6. 防火墙设置
```bash
# 如果使用 UFW
sudo ufw allow 20/tcp
sudo ufw allow 21/tcp
sudo ufw allow 40000:50000/tcp
```

### 7. 测试连接
```bash
# 连接信息
主机：your_server_ip
端口：21
用户名：UserAAA（或其他用户名）
密码：对应设置的密码
```

### 8. 用户管理命令
```bash
# 添加新用户
sudo bash create_ftp_user.sh "UserDDD" "passwordDDD"

# 修改用户密码
sudo passwd UserAAA

# 删除用户
sudo userdel -r UserAAA
```

### 9. 监控和维护
```bash
# 查看当前连接
sudo netstat -nat | grep :21

# 查看日志
sudo tail -f /var/log/vsftpd.log

# 查看目录权限
ls -la /home/ftp_root/
```

### 安全建议：

1. 文件权限设置
```bash
# 定期检查权限
find /home/ftp_root -type d -exec chmod 755 {} \;
find /home/ftp_root -type f -exec chmod 644 {} \;
```

2. 用户限制
```conf
# 在 vsftpd.conf 中添加
max_clients=50
max_per_ip=5
```

3. 定期备份
```bash
# 创建备份脚本
#!/bin/bash
backup_date=$(date +%Y%m%d)
tar -czf /backup/ftp_backup_$backup_date.tar.gz /home/ftp_root/
```

这个配置：
- 每个用户都有独立的主目录
- 用户被限制在自己的目录中
- 包含所有需要的子目录
- 权限设置合理
- 易于管理和维护
