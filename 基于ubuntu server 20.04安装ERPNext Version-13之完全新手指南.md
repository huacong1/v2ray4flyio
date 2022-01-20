	步骤如下

1. 准备一台装好ubuntu20.04的服务器，本地虚拟机或者云服务器皆可，虚拟机的网络要用桥接模式。Ubuntu 20.04server安装会顺利些。

2. 设置root密码（虚拟机、物理机、腾讯云需要做这一步，阿里云在初始化镜像时设置的是root用户和密码，不用做这一步，其他未测试，需视情况而定。）

   ```markdown
   sudo passwd root
   ```

3. 添加用户（虚拟机、物理机、腾讯云可以略过这一步，其中虚拟机和物理机可以使用安装系统时建的用户，腾讯云可以用ubuntu用户 ，阿里云需要做这一步）01

   ```markdown
   adduser frappe
   #用户名USERNAME换成自己计划使用的用户名
   ```

4. 将用户设置sudo权限：（如跳过第3步则这步也可以跳过）

   ```markdown
   usermod -aG sudo frappe
   #记得把命令行里的USERNAME改成自己刚设置的用户名
   ```

5. 通过ssh登录root用户

   ```markdown
   #虚拟机、物理机和腾讯云需要先设置将端口22开放才能通过ssh连接
   sudo vim /etc/ssh/sshd_config
   
   按i，移动光标打到以下内容修改：
   
   #Port 22
   改为
   Port 22
   
   #PermitRootLogin prohibit-password
   改为
   PermitRootLogin yes
   
   修改好按ESC,shift+:，输入wq回车
   
   重启服务
   /etc/init.d/ssh restart
   ```

   

6. 替换国内镜像源，下面用的是清华的镜像源，操作如下(root用户可不输入sudo，国内的云服务器都预置有各自的源镜像，可以不用做这一步)：

   ```markdown
   1.备份
   sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak 
   
   2.修改
   sudo vim /etc/apt/sources.list
   
   3.按ggVG进行全选，按d进行删除
   4.将下面源粘贴
   5.按esc，再按shift+:,输入wq回车(这步是保存退出)
   ```

   ```markdown
   deb http://mirrors.aliyun.com/ubuntu hirsute main restricted
   deb http://mirrors.aliyun.com/ubuntu hirsute-updates main restricted
   deb http://mirrors.aliyun.com/ubuntu hirsute universe
   deb http://mirrors.aliyun.com/ubuntu hirsute-updates universe
   deb http://mirrors.aliyun.com/ubuntu hirsute multiverse
   deb http://mirrors.aliyun.com/ubuntu hirsute-updates multiverse
   deb http://mirrors.aliyun.com/ubuntu hirsute-backports main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu hirsute-security main restricted
   deb http://cn.archive.ubuntu.com/ubuntu hirsute-security universe
   deb http://cn.archive.ubuntu.com/ubuntu hirsute-security multiverse
   ```
   
7. 将pip源配置为国内源(root用户和安装erpnext的用户都分别配置)

   创建pip.config文件

   ```markdown
   mkdir ~/.pip
   vim ~/.pip/pip.conf
   ```

   添加pip源

   ```markdown
   [global]
   index-url = http://mirrors.aliyun.com/pypi/simple/
        
   [install]
   trusted-host=mirrors.aliyun.com
   ```

   配置完按Esc键，shift+:，输入wq回车.

   ```
   sudo mkdir /root/.pip
   
   sudo cp ~/.pip/pip.conf /root/.pip
   ```

   

8. 更新并重启  02

   ```markdown
   apt update && apt upgrade -y && shutdown -r now
   ```

9. 下载node.js     03

   ```markdown
   curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
   #有时候运行完没反应，直接跳回命令输入，这时候要再运行到出现以下提示
   ## Installing the NodeSource Node.js 12.x repo...
   ```

10. 安装操作系统级依赖     04

   ```markdown
   apt install -y nodejs mariadb-server-10.3 redis-server python3-pip nginx python3-testresources git libffi-dev python3-dev libssl-dev gcc g++ make
   ```

11. 用nano编辑my.cnf文件   05

   ```markdown
   nano /etc/mysql/my.cnf
   ```

   将光标移动到最后空白行，复制以下文本内容，ctrl + O,回车确认，ctrl + X返回命令行（注意格式一定要按下方的格式，如内容换行不对，到12步时会过不了。）

   ```markdown
   [mysqld]
   character-set-client-handshake = FALSE 
   character-set-server = utf8mb4 
   collation-server = utf8mb4_unicode_ci 
   
   [mysql]
   default-character-set = utf8mb4
   ```

12. 重启sql    06

   ```markdown
   service mysql restart
   ```

13. mysql的安全配置   07

   ```markdown
   mysql_secure_installation
   ```

   第一个输入数据库密码对话框出来的时候，直接敲回车代表没有密码，剩下的按照下面选择：	

   ```markdown
   Enter current password for root (enter for none): 
   #这里直接回车
   Set root password? [Y/n] Y
   New password:  #输入数据库密码
   Re-enter new password: #重复输入数据库密码
   Remove anonymous users? [Y/n] Y
   Disallow root login remotely? [Y/n] n
   Remove test database and access to it? [Y/n] Y
   Reload privilege tables now? [Y/n] Y
   ```

14. 输入上面新设置的数据库root账号密码，进入数据库命令行，并执行下面的语句     08

    ```markdown
    mysql -u root -p
    
    USE mysql; 
    UPDATE user SET plugin=' ' WHERE user ='root'; 
    FLUSH PRIVILEGES;
    exit;
    ```

15. ### 【重要】关闭ssh终端，重新以自己“新创建的用户名”和密码登录（参考第3、4步，如果没有新建的用户，则用安装系统时的用户进行登录，如阿里云才需要用第3、4步建好的用户来登录）

16. 安装yarn    09

    ```markdown
    sudo npm install -g yarn
    
    #保险起见，还需要将刚刚的yarn,node,npm添加运行权限。 sudo chmod +x /usr/bin/node. 默认有运行权限。——Jason Zhang
    #配置Yarn的源：
    
    yarn config get registry查看源, 如果不是淘宝的源就切换为淘宝的源
    
    yarn config set registry https://registry.npm.taobao.org
    yarn config set sass_binary_site "https://npm.taobao.org/mirrors/node-sass/"
    yarn config set phantomjs_cdnurl "http://cnpmjs.org/downloads"
    yarn config set electron_mirror "https://npm.taobao.org/mirrors/electron/"
    yarn config set sqlite3_binary_host_mirror "https://foxgis.oss-cn-shanghai.aliyuncs.com/"
    yarn config set profiler_binary_host_mirror "https://npm.taobao.org/mirrors/node-inspector/"
    yarn config set chromedriver_cdnurl "https://cdn.npm.taobao.org/dist/chromedriver"
    npm config set phantomjs_cdnurl https://npm.taobao.org/mirrors/phantomjs/
    
    ```
  
17. 查看版本，对照一下，这一步不做也行    010

    ```markdown
    node -v && npm -v && python3 -V && pip3 -V && yarn -v
    ```

18. 安装bench，即erpnext系统的命令行管理工具，类似windows系统的程序管理器。    011

    ```markdown
    sudo pip3 install frappe-bench
    #2021-11-13加多一个 -H 未验证
    #-H 将环境变数中的 HOME （家目录）指定为要变更身份的使用者家目录（如不加 -u 参数就是系统管理者 root ）
    ```

19. 安装git，下一步bench init可能会报错缺少git。    012

    ```markdown
    sudo apt install git
    ```

20. 使用bench命令安装frappe框架，记得把frappe-bench（下方的version-13后面的名字）改成自己想要的名字，这一步时间比较长，别着急，代码库已经加了码云地址参数。如果网络超时失败，可重新运行该命令，重新运行之前需使用命令 `rm -r frappe-bench` 删除之前生成的目录。

    ```markdown
    bench init --frappe-branch version-13 frappe-bench --frappe-path=https://gitee.com/phipsoft/frappe
    ```

    ```markdown
    #下面这段是警告说pip有新版本，可忽略
    WARNING: You are using pip version 21.0.1; however, version 21.1 is available.
    You should consider upgrading via the '/home/frappe/frappe-bench/env/bin/python -m pip install --upgrade pip' command.
    
    #这步报错的时候
    pip._vendor.urllib3.exceptions.ReadTimeoutError: HTTPSConnectionPool(host='files.pythonhosted.org', port=443): Read timed out.
    #可以先安装下面这个
    pip install library -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
    ```

    下载指定版本，将上面的version-13改为仓库中的标签名称，如v13.15.1：

    ```
    bench get-app --branch v13.15.1 erpnext https://gitee.com/qinyanwan/frappe
    ```

    

21. 进入bench目录

    ```markdown
    cd frappe-bench
    ```

22. 新建站点，名字自己取，安装时会提示输入数据库root账号的密码, 新站点数据库及erp系统管理员账号administator 密码，其中数据库root账号密码须与上述数据库安装时密码一致，其它密码自己取 --db-password xxx 也可以命令行参数中直接输入好密码，--mariadb-root-password yyyy --admin-password zzzz

    ```markdown
    bench new-site mysite
    #新建的站点名为mysite
    ```

    

23. 下载erpnext

    ```
    bench get-app --branch version-13 erpnext https://gitee.com/phipsoft/frappe
    ```

    下载指定版本，将上面的version-13改为仓库中的标签名称，如v13.15.1：

    ```
    bench get-app --branch v13.15.1 erpnext https://gitee.com/phipsoft/frappe
    ```

    

24. 安装erpnext

    ```markdown
    bench install-app erpnext
    ```

25. 设置为生产环境，即用supervisorctl管理所有进程，使用nginx做反向代理，USERNAME换成第3步新建的账号，大功告成。

    ```markdown
    sudo bench setup production USERNAME
    #重要：设置成生产环境后，不用执行bench start进行启动！！！
    ```

26. 安装完后可查看一下是否有活动的wokers

    ```markdown
    bench doctor
    
    #正常情况下会显示如下：
    -----Checking scheduler status-----
    Scheduler disabled for erpnext
    Scheduler inactive for erpnext
    Workers online: 3
    -----erpnext Jobs-----
    ```

27. 以上完成后查看一下安装了哪些app

    ```
    bench version 
    ```

    正常会显示以下两个app

    ```
    erpnext 13.x.x
    frappe 13.x.x
    ```

    

28. 如果是虚似机安装，需要防火墙入站打开80端口。

    ```markdown
    ufw allow 80
    
    sudo ufw status verbose
    
    #打开后显示如下：
    root@erpnext:~# ufw allow 80
    Rule added
    Rule added (v6)
    root@erpnext:~# sudo ufw status verbose
    Status: active
    Logging: on (low)
    Default: deny (incoming), allow (outgoing), deny (routed)
    New profiles: skip
    
    To                         Action      From
    --                         ------      ----
    22/tcp                     ALLOW IN    Anywhere                  
    22                         ALLOW IN    Anywhere                  
    80                         ALLOW IN    Anywhere                  
    22/tcp (v6)                ALLOW IN    Anywhere (v6)             
    22 (v6)                    ALLOW IN    Anywhere (v6)             
    80 (v6)                    ALLOW IN    Anywhere (v6) 
    ```

29. 阿里云服务器要在安全组里添加80的入站端口号。

    ```markdown
    您将需要在ERPNext服务器上打开以下端口：
    
    80/tcp and 443/tcp for HTTP and HTTPS respectively
    
    HTTP和HTTPS分别为80/tcp和443/tcp
    
    3306/tcp for MariaDB connection (recommended only if you need remote access to database)
    
    3306/tcp用于MariaDB连接(仅在需要远程访问数据库时才建议使用)
    
    143/tcp and 25/tcp for IMAP and STMP respectively
    
    IMAP和STMP分别为143/tcp和25/tcp
    
    22/tcp for SSH (if you have not already enabled OpenSSH in your UFW settings)
    
    SSH 22/tcp (如果尚未在UFW设置中启用OpenSSH )
    
    8000/tcp for testing your platform before deploying to production
    
    8000/tcp用于在部署到生产环境之前测试平台
    
    #要一次打开多个端口，可以使用以下命令：
    
    sudo ufw allow 22,25,143,80,443,3306,8000 /tcp
    sudo ufw allow 22,25,143,80,443,3306,8000 /tcp
    
    现在确认防火墙的状态：
    
    sudo ufw status
    ```

    

**附注：余老师开发的本地化应用**

**安装中国本地化APP**

1. 获取APP

   ```
   bench get-app https://gitee.com/yuzelin/erpnext_chinese.git
   ```

2. 安装APP

   ```
   bench install-app erpnext_chinese
   ```

3. 用浏览器(推荐 Chrome或Firefox)输入IP或域名，登录系统，用户名administrator,密码是第21步admin-password密码。

**安装开箱即用应用**

1. 获取APP

   ```
   bench get-app https://gitee.com/yuzelin/erpnext_oob.git
   ```

2. 安装APP

   ```
   bench install-app erpnext_oob
   ```

3. 

**常见问题**

1. wkhtmltopdf安装时出现字体库依赖无法安装等错误 参考了这个https://askubuntu.com/questions/604029/dependency-problems-with-wkhtmltopdf-when-trying-to-install-latest-version，最后是以下命令解决了 

   ```
   sudo apt-get -f install
   ```

   

2. 通过打印转PDF时出现乱码 参考这个帖子https://www.taodudu.cc/news/show-1772808.html，用以下命令安装字体

   ```
   sudo apt-get install ttf-wqy-zenhei -y
   sudo apt-get install ttf-wqy-microhei -y
   ```

#### Ubuntu20.04 安装打印wkhtmltopdf 库

1、先下载适合我们系统的安装包并进行安装：

```
wget "https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.focal_amd64.deb" -O /tmp/wkhtml.deb
```

2、下一步进行安装

```
sudo dpkg -i /tmp/wkhtml.deb
```

3、这时可能会显示缺少依赖的错误，以下命令可解决这一问题：

```
sudo apt -f install
```

4、现在，我们可以检查wkhtmltopdf 库是否正确安装并确认是否为所需版本：

```
wkhtmltopdf –version
```

##### 显示wkhtmltopdf 0.12.6 (with patched qt)即是正确版本

其他注意：

```
apt install wkhtmltopdf
```

##### 这条命令国内阿里云源自动安装版本0.12.5，非patched qt 版本，erpnext 不打印页面头部和底部。

如果wkhtmltopdf 库不是我们需要的版本，应对其进行卸载，命令如下：

```
sudo apt remove --purge wkhtmltopdf
```



#### 腾讯云安装完后不能正常显示初始化界面的处理方法

```markdown
依次执行下面三条命令，通常只需要执行第三条，修改frappe user的权限就可以。
bench build
~/frappe-bench/sites$ chmod a+x assets
chmod 701 /home/<frappe-user>
```

