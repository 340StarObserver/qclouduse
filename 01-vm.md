### 虚拟机 ###


#### 1. 系统盘的选用 ####

        本地盘 : 延时小，无备份，不能调整CPU和内存
        云硬盘 : 延时大，三副本，可以调整CPU和内存


#### 2. 数据盘的选用 ####

        // 比如数据库服务的日志和数据建议存储到不同的硬盘上
        HDD   : 存日志
        SSD   : 存数据


#### 3. 更改ssh的端口 + 普通用户登入 ####

        vi /etc/ssh/sshd_config
            // 更改 Port 35628
        
        useradd -d /home/wuanjun -m wuanjun
        passwd wuanjun
        groupadd common
        usermod -g common wuanjun
            // 创建用户wuanjun，并创建主目录，它的组是common
        
        vi /etc/ssh/sshd_config
            // 在 DenyUsers  中添加 root
            // 在 AllowUsers 中添加 wuanjun
            // 即，允许用户wuanjun以ssh方式登入，拒绝用户root
            // 优先级为 : DenyUsers > AllowUsers > DenyGroups > AllowGroups
        
        service sshd restart
            // 重启ssh


#### 4. 配置iptables ####

        systemclt stop firewalld
        systemctl mask firewalld
            // 关闭默认的防火墙 
        
        yum install iptables
        yum install iptables-services
            // 安装iptables作为防火墙
        
        systemctl enable iptables
        systemctl start iptables
            // 设置iptables开启启动，并启动之
        
        vi /etc/sysconfig/iptables
            // 配置iptables规则
            -A INPUT -i lo -j ACCEPT
            -A INPUT -p icmp -j ACCEPT
            -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
            -A INPUT -p tcp --dport 35628 -j ACCEPT
            -A INPUT -j REJECT
        
        service iptables restart
            // 重启iptables，使配置文件里的规则生效
        
        iptables -L
            // 查看新规则列表，确认生效
