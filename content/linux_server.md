+++
title = "Oracle Cloud Linux Server Configuration"
date = 2022-05-02
draft = false
+++


## Oracle Cloud Linux Server 基础配置
1. 通过密码登录 SSH
   - 切换到 root 用户: `sudo su`
   - 设置 root 密码: `passwd`
   - 修改 sshd_config: `vim /etc/ssh/sshd_config`, PermitRootLogin 修改为 yes, PasswordAuthentication 修改为 yes
   - 重启 sshd: Oracle linux 使用 `systemctl restart sshd`, Ubuntu 使用 `/etc/init.d/ssh restart`

2. 添加用户
   - create new user: `sudo adduser username`
   - change user password: `passwd username`
   - switch to user: `su username`
3. v2ray 配置
   - 使用一键脚本: 
     - `bash <(curl -sL https://cdn.jsdelivr.net/gh/Misaka-blog/Xray-script@master/xray.sh)`
     - arm 服务器使用: `bash <(curl -s -L https://git.io/v2ray.sh)`
   - cloudFlare worker: 
     - 新建一个 worker, 脚本内容为
        ```javascript
        addEventListener(
            "fetch",event => {
                let url=new URL(event.request.url);
                url.hostname="arm.xiongdong.xyz";
                let request=new Request(url,event.request);
                event. respondWith(
                fetch(request)
                )
            }
        )
        ```
    - 测试 Send，返回 200OK
    - 复制 Worker地址，填入到 V2ray 伪装域名（Host）中
    - 地址可以修改为优选的 CloudFlare IP
4. syncthing
   - 安装
      - 添加GPG
          `
          sudo apt-get install curl && 
          curl -s https://syncthing.net/release-key.txt | sudo apt-key add -
          `
      - 添加源
          `
          echo "deb https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list
          `
      - 安装apt-transport-https
          `
          sudo apt-get install apt-transport-https
          `
      - 更新&安装
          `
          sudo apt-get update && 
          sudo apt-get install syncthing
          `
   - 配置 

        修改 `/home/ubuntu/.config/syncthing/config.xml`, 找到:
        ```xml
        <gui enabled="true" tls="false" debugging="false">
            <address>127.0.0.1:8384</address>
        ```
        tls 修改为 "true", 127.0.0.1 修改为 0.0.0.0 
   - 开放端口

        `
        apt-get install iptables && iptables -I INPUT -p tcp --dport 8384 -j ACCEPT && iptables-save
        `
        上述规则持久化(正常重启后会失效):
        ```bash
        apt-get install iptables-persistent
        netfilter-persistent save
        netfilter-persistent reload
        ```
5. transmission
    - 安装
        `
        sudo apt-get install transmission-daemon
        `
    - 配置
  
        修改 `vim /var/lib/transmission-daemon/.config/transmission-daemon/settings.json`，参照:
        ```json
        "rpc-authentication-required": true,
        "rpc-bind-address": "0.0.0.0",
        "rpc-enabled": true,
        "rpc-host-whitelist": "",
        "rpc-host-whitelist-enabled": true,
        "rpc-password": "123456",
        "rpc-port": 9091,
        "rpc-url": "/transmission/",
        "rpc-username": "admin",
        "rpc-whitelist": "",
        "rpc-whitelist-enabled": false,
        ```
    - 开放9091端口，参照上述 syncthing 的方式
    - 启动 `service transmission-daemon start`   
