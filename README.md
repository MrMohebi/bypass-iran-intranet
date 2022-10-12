# bypass-iran-intranet
setup Proxy chains to bypass internet blockade in iran



you need two servers one in an Iranian data center and one outside (preferably Europe because of latency). Use cloud servers, thay are 
cheap and also easy to use. Order the less config is possible. even 1 core of cpu and 512M of ram are completely enough.

we consider iran server IP is: ```103.130.0.10```

european IP: ```102.128.0.20```

also iran and europe both ports is: ```2424```

## Europe server config
First, setup an shadowsocks proxy on the european server. the easy way is to set it up with docker-compose. 
v2ray plugin should be enabled to protect proxy to be banned by protocol.

To installing docker and docker compose on your server, use this instruction [install docker](https://docs.docker.com/engine/install/ubuntu/).

after installation. create directory that contains these two files([docker-compose.yml](#rootshadowdocker-composeyml) and [config.json](#rootshadowconfigjson))

#### ```/root/shadow/docker-compose.yml```
```
version: '3.0'
services:
  shadow-v2ray:
    image: teddysun/shadowsocks-libev
    restart: always
    container_name: shadow-v2ray
    ports:
      - 2424:1080/udp
      - 2424:1080
    volumes:
      - ./config.json:/etc/shadowsocks-libev/config.json
```

#### ```/root/shadow/config.json```
```
{
    "server":"0.0.0.0",
    "server_port":1080,
    "password":"passW0rd123",
    "timeout":300,
    "method":"aes-256-gcm",
    "fast_open":false,
    "nameserver":"8.8.8.8",
    "plugin":"v2ray-plugin",
    "plugin_opts":"server"
}
```
Change password as you wish. other config settings can be changed or stay as thay are.


 then go to your dirctory and run composer up(in our example)
 ```bash
 $ cd /root/shadow/
 $ docker compose up -d
 ```


## Iran server
Due to sanctions you can't install docker easily in iranian servers. so instead of dockerizing nginx, we just install it as old fashion. As I've heard some people had nginx problems with Arvancloud servers. I think its because Arvan uses its own mirror ubuntu repositories instead of orginal one. just consider you may have problem with Arvan servers   


install nginx:
```bash
$ sudo apt update
$ sudo apt install -y nginx 
```

then edit nginx config file to redirect all incoming requests to european server.

change IP to your european server ip and run this command

``` bash
$ sudo cat >> /etc/nginx/nginx.conf <<EOF

stream {
    server {
        # iran PORT
        listen     2424;
        # europe IP:PORT
        proxy_pass IP:2424;
    }

}
EOF
```

restart nginx to changes be applied 
```bash
$ sudo systemctl restart nginx
```


And we are done! enjoy freedom :)

To connect to this proxy use [below instructions](#config-clients)

## config clients

### Android clints:
for android devices we need these two app:
- [V2ray Plugin](https://play.google.com/store/apps/details?id=com.github.shadowsocks.plugin.v2ray)
- [Shadowsocks](https://play.google.com/store/apps/details?id=com.github.shadowsocks)

create new proxy then set "Remote Port, Password, Encrypt Method(in this example 'aes-256-gcm')" base on [```config.json```](#rootshadowconfigjson) file JUST remmeber put Iran ip for **Server** section in our example it's 103.130.0.10. also at bottom of page set plugin as v2ray


### Linux clients
you can use ```ss-local``` find more on [its docs](https://github.com/shadowsocks/v2ray-plugin)



# considerable issues:

### equal downloading and uploading
today I've heard data centers are measuring your upload and download. if thay be in same rage, detect your server as VPN so thay block it.
to protect your server from bieng banned, you can regularly download dummy file and delete it. to achive this:

- on your IRAN serve create file named ```dummydl.sh```
``` bash
$ cat > /root/dummydl.sh <<EOF
#!/bin/bash
wget -c "https://speed.hetzner.de/1GB.bin" -O /dev/null
EOF
```
- set execution permission
```bash
$ chmod +x dummydl.sh
```
- add this shell file to cronjobs
```bash
$ crontab -e
```
and append this line at the end of file

```0 */6 * * * sh /root/dummydl.sh```

# pro tips

### half traffic calculation:
put your Iran server behind Arvan DNS, then when you use proxy, traffic usage will be calculated in half :)
