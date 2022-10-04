# bypass-iran-intranet
setup Proxy chains to bypass internet blockade in iran



you need two servers one in an Iranian data center and one outside (preferably Europe because of latency).

we consider iran server IP is: ```103.130.0.10```

european IP: ```102.128.0.20```

also iran and europe both ports is: ```2424```

## Europe server config
First, setup an shadowsocks proxy on the european server. the easy way is to set it up with docker-compose. 
v2ray plugin should be enabled to protect proxy to be banned by protocol

#### ```docker-compose.yml```
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

#### ```config.json```
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
 then run composer 
 ```bash
 $ docker-compose up -d
 ```


## Iran server
install nginx:
```bash
$ sudo apt update
$ sudo apt install -y nginx 
```

then edit nginx config file to redirect all incoming requests to european server.

add this block above of ```http``` block

#### ```/etc/nginx/nginx.conf```
```
stream {
    server {
        # iran PORT
        listen     2424;
        # europe IP:PORT
        proxy_pass 102.128.0.20:2424;
    }

}

```
restart nginx to changes be applied 
```bash
$ sudo systemctl restart nginx
```


And we are done!
## Android clints:
for android devices we need these two app:
- [V2ray Plugin](https://play.google.com/store/apps/details?id=com.github.shadowsocks.plugin.v2ray)
- [Shadowsocks](https://play.google.com/store/apps/details?id=com.github.shadowsocks)

create new proxy then set information base on [```config.json```](https://github.com/MrMohebi/bypass-iran-intranet#configjson) file JUST remmeber put iran ip for IP section in our example it's 103.130.0.10. also set plugin as v2ray


## Linux clients
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
$ chmod +x dl.sh
```
- add this shell file to cronjobs
```bash
$ crontab -e
```
and append this line at the end of file

```0 */6 * * * sh /root/dummydl.sh```
