# 多县长mTalker FTP设置

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
    sudo mkdir -p "/home/ftp_root/FTP_$USERNAME/UploadMsg_FromWeMole"
    sudo mkdir -p "/home/ftp_root/FTP_$USERNAME/Download_ToSendingOut"
    sudo mkdir -p "/home/ftp_root/FTP_$USERNAME/tmp"

    # 设置所有权
    sudo chown -R $USERNAME:$USERNAME "/home/ftp_root/FTP_$USERNAME"

    # 设置权限
    sudo chmod 755 "/home/ftp_root/FTP_$USERNAME"
    sudo chmod 755 "/home/ftp_root/FTP_$USERNAME/UploadMsg_FromWeMole"
    sudo chmod 755 "/home/ftp_root/FTP_$USERNAME/Download_ToSendingOut"
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






# WhatsApp群接收消息保存到文件夹  
GitHub仓库名：WhatsAppGroup_to_Folder
  
## 需求实现：把某WhatsApp群的每一条消息（如是英语，则翻译为中文）接收后，按指定格式保存到某文件夹的一个文本文件中，如是图片（或表情图或附件文件）也存到文件。

## WhatsApp接入方案：
最佳推荐：WhatsApp Business API
原因：
- 接收消息完全免费
- 最稳定可靠
- 没有中间商增加的延迟
- 长期使用最经济

### 其他方案对比
WhatsApp Business API:
- 接收消息：免费
- 仅企业认证等前期成本

MessageBird:
- 接收消息：约 $400/月
- 无需认证费用

Twilio:
- 接收消息：约 $500/月
- 无需认证费用

Vonage:
- 接收消息：约 $500/月
- 无需认证费用


## 技术细节
编程语言不限，但运行服务器环境为：Debian11x64 1G内存
内存较小，且还有其他App在运行。
运行服务器上已安装有Python3.
因此实现方案需考虑：占用内存不能超过200M，最好小于100M

翻译使用Google的免费翻译服务API 或 OpenAI的LLM免费版（如gpt-4o-mini）或 其他质量较高的翻译方案

文本与图片等 都保存到：Dir-SendingOut

图片（或表情图或附件文件）的文件名中会标识 发送者的昵称

文件名如：序号（如用UNIX-TIMESTAMP）-From:WhatsApp_GroupName-To:发送者的昵称.jpg
序号用于标识消息的先后次序。如：

1728908628_Picture_From:WhatsAppGroup@@GroupName@发送者的昵称.JPEG

17289036538_Text_From:WhatsAppGroup@@GroupName@发送者的昵称.TXT






# 清理现有mTalker（Python项目）中的大量无用代码（实际运行时根本不会用到），减少运行时的内存消耗。同时保持正常运行
记录优化清理前的内存消耗，并对比优化后的内存消耗。


# 功能增加：将mTalker从单县长（单微信号）改为多县长（多微信号）



# 功能增加：使每个聊天室可以设置自己专用的系统提示词、诱导回答语



# 功能增加：增加　Grok、Google Gemini作为回答的GPT引擎



# 国产GPT合并到mTalker，改为用 weMole + mTalker



# 各县长、各群 的负责人 可以自定：系统提示词、诱导回答语


