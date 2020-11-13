Nginx setup configuration
===================

## Content
- [Requirements](#requirements)
- [Installation](#installation)
- [Starting and configuring the service](#configuring-service)
- [Facebook rtmps support](#fb-rtmps-support)
- [Troubleshooting](#troubleshooting)
- [Links](#links)

### Requirements <a name="requirements" />

- Git, wget, gcc, openssl, stunnel
- Repository [nginx-rtmp-module](https://github.com/arut/nginx-rtmp-module.git)
- nginx server [nginx](https://nginx.org)

### Installation <a name="installation" />

```bash
# Install requirements

wget https://raw.githubusercontent.com/q3aql/ffmpeg-install/master/ffmpeg-install;
chmod a+x ffmpeg-install;
./ffmpeg-install --install release;
yum install -y git wget make mlocate epel-release;
yum install -y gcc pcre2-devel pcre pcre-devel openssl openssl-libs openssl-devel;

## Clone the repository
git clone https://github.com/arut/nginx-rtmp-module.git;

## Download nginx server. Latest version when writing this document 1.18.0
NGINX_VERSION=1.18.0;
wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz;
tar -xzf nginx-${NGINX_VERSION}.tar.gz;
## Configuring rtmp module
cd nginx-${NGINX_VERSION} && ./configure --add-module=/root/nginx-rtmp-module/ --with-http_ssl_module;

./configure --add-module=/root/nginx-rtmp-module --with-http_ssl_module --with-debug --with-cc-opt="-Wimplicit-fallthrough=0"

## Validate nginx works
/usr/local/nginx/sbin/nginx -v

```

### Starting and configuring the service <a name="configuring-service" />
```bash
vi /usr/local/nginx/conf/nginx.conf  ## look the nginx.config file

## Validate nginx works
/usr/local/nginx/sbin/nginx -v
## Start service
/usr/local/nginx/sbin/nginx
## Stop service
/usr/local/nginx/sbin/nginx -s <stop, quit, reopen, reload>
## Test configuration and exit
/usr/local/nginx/sbin/nginx -t
## Help
/usr/local/nginx/sbin/nginx -h

# Set your nginx.conf file
```

### Facebook rtmps support <a name="fb-rtmps-support" />
use stunnel (stunnel)[https://sites.google.com/view/facebook-rtmp-to-rtmps/home]

```bash
# Create the files
tocuh /etc/default/stunnel4
tocuh /etc/stunnel/stunnel.conf

Create the dirs
- mkdir -p /var/run/stunnel/
- mkdir -p /var/log/stunnel/
```

`vim /etc/default/stunnel4`
```
ENABLE=1
```

`vim /etc/stunnel/stunnel.conf`
```vi
pid = /var/run/stunnel/stunnel.pid
output = /var/log/stunnel/stunnel.log

#setuid = stunnel
#setgid = stunnel

# https://www.stunnel.org/faq.html
socket = r:TCP_NODELAY=1
socket = l:TCP_NODELAY=1

debug = 4

[fb-live]
client = yes
accept = 1936
connect = live-api-s.facebook.com:443
verifyChain = no
```

Start and enable services
```
sudo systemctl enable stunnel.service
sudo systemctl restart stunnel.service
```

### Troubleshooting <a name="troubleshooting" />

Possible errors during installation **Error**
```bash
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g   -I src/core -I src/event -I src/event/modules -I src/os/unix -I /root/nginx-rtmp-module/ -I objs -I src/http -I src/http/modules \
    -o objs/addon/nginx-rtmp-module/ngx_rtmp_eval.o \
    /root/nginx-rtmp-module//ngx_rtmp_eval.c
/root/nginx-rtmp-module//ngx_rtmp_eval.c: In function 'ngx_rtmp_eval':
/root/nginx-rtmp-module//ngx_rtmp_eval.c:160:17: error: this statement may fall through [-Werror=implicit-fallthrough=]
                 switch (c) {
                 ^~~~~~
/root/nginx-rtmp-module//ngx_rtmp_eval.c:170:13: note: here
             case ESCAPE:
             ^~~~
cc1: all warnings being treated as errors
make[1]: *** [objs/Makefile:1349: objs/addon/nginx-rtmp-module/ngx_rtmp_eval.o] Error 1
make[1]: Leaving directory '/root/nginx-1.18.0'
make: *** [Makefile:8: build] Error 2
```
**Solution**
```bash
./configure --add-module=/root/nginx-rtmp-module --with-http_ssl_module --with-debug --with-cc-opt="-Wimplicit-fallthrough=0"
```

Stunnel can't start. Can't open/read log file
**Solution**
```bash
ausearch -c 'stunnel' --raw | audit2allow -M my-stunnel

semodule -X 300 -i my-stunnel.pp
```

### Links <a name="links" />
- [Install ffmpeg](https://www.binarycomputer.solutions/install-ffmpeg-centos-6-centos-7/)
- [Can not strem to Twitter](https://github.com/arut/nginx-rtmp-module/issues/898)
- [Other solution for multistreaming to check](https://github.com/michaelkamprath/multi-service-rtmp-broadcaster)
- [Tutorial facebook-rtmp-to-rtmps](https://sites.google.com/view/facebook-rtmp-to-rtmps/home)
- [Tutorial alibabacloud](https://www.alibabacloud.com/blog/how-to-install-nginx-with-rtmp-module-on-centos-7_594403)
- [Github arut nginx-rtmp-module](https://github.com/arut/nginx-rtmp-module)
- [nginx-rtmp blog ](http://nginx-rtmp.blogspot.com/)
- [opensource blog ](https://opensource.com/article/19/1/basic-live-video-streaming-server)
- [Tutorial Video](https://www.youtube.com/watch?v=Y-9kVF6bWr4)
