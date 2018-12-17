---
layout: post
title:  "ubuntu下配置shadowsocks"
date:   2018-12-17 09:30:00 +0800
categories: 文章
---

ubuntu下很容易配置shadow socks，直接拷贝以下脚本，把以下脚本中的“YourPassword”改成自己需要的密码，执行脚本即可。

```
#!/bin/sh
# copy this script content to a init.sh file, and run "sudo bash init.sh"

# config bbr
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr

# create ss config file
cat <<EOF > /etc/shadowsocks.json
{
"server":"0.0.0.0",
"server_port":8838,
"local_address":"127.0.0.1",
"local_port":1080,
"password":"YourPassword",
"timeout":600,
"method":"aes-256-cfb"
}
EOF

# create service file, start shadowsocks on system start
cat <<EOF > /etc/init.d/shadowsocks
#!/bin/sh
### BEGIN INIT INFO
# Provides: shadowsocks
# Required-Start: $remote_fs $syslog
# Required-Stop: $remote_fs $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: start shadowsocks
# Description: start shadowsocks
### END INIT INFO
start(){
    #ssserver -c /etc/ss-conf.json -d start
    /usr/bin/python /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start
}
stop(){
    #ssserver -c /etc/ss-conf.json -d stop
    /usr/bin/python /usr/local/bin/ssserver -c /etc/shadowsocks.json -d stop
}

case "\$1" in
start)
　　　start
　　　;;
stop)
　　　stop
　　　;;
reload)
　　　stop
　　　start
　　　;;
*)
　　　echo "Usage: $0 {start|reload|stop}"
　　　exit 1
　　　;;
esac
EOF

chmod +x /etc/init.d/shadowsocks

cat <<EOF > /etc/init/shadowsocks.conf
start on (runlevel [2345])
stop on (runlevel [016])
pre-start script
/etc/init.d/shadowsocks start
end script
post-stop script
/etc/init.d/shadowsocks stop
end script
EOF

update-rc.d shadowsocks defaults
systemctl daemon-reload

# start ss service
service shadowsocks restart
```

