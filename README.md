# Music-LAN-Server
A repo to document my Music LAN Server.

Prerequisites- Linux Operating System and a general understanding of how it work, OBS (Open Broadcaster Software) and a general understanding of how it works, VLC Media Player (For my uses I use VLC on iOS(iPhone 6), and desktop)

I am going to write this guide with a few different perspectives.

One that is a quick Arch Linux method.

And Another that MIGHT(But should) work for other Linux Distros if you have the required files.

Perhaps this can be done with Windows or MacOS if you searched into it but that is note the scope of this guide.

The configurations I have put here are pretty slimmed down compared to others that I had seen while experimenting with this.

The quick Arch Linux guide is more extensively documented as it is my method but in case anything ever happens I am including the second guide in case I have to ever install nginx from source to create a server.

I did a lot of searching online to get this to work and this guide is pretty dirty so hopefully you can get it to work too :) 

Enjoy~

Links
-----
- https://aur.archlinux.org/packages/nginx-rtmp/
- https://www.servermania.com/kb/articles/nginx-rtmp/
- https://nginx-rtmp.blogspot.com/

Quick Arch Linux Method
------------------------

1. Use your AUR manager to download and install nginx-rtmp 1.18.0-3 (https://aur.archlinux.org/packages/nginx-rtmp/)

2. Use your terminal and go to /etc/nginx

3. Add the following configuration to your nginx.conf

```
#user html;                   
worker_processes  auto;                                          
                                
#error_log  logs/error.log;
#error_log  logs/error.log  notice;                                                                                                
#error_log  logs/error.log  info;
                                
#pid        logs/nginx.pid;                                      
                                

events {                                                                                                                           
    worker_connections  1024;
}                           
                                                                 
rtmp{                                                            
        server{                                                  
                listen 1935;                                                                                                       
                chunk_size 4096;           
                application live{
                        live on; 
                        record off;                                                                                                
                }                                                
        }
}                          
                                
http {    
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '  
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
                                
    #access_log  logs/access.log  main;
                                                                 
    sendfile        on;
    #tcp_nopush     on;
                                
    #keepalive_timeout  0;                                       
    keepalive_timeout  65;
                                
    #gzip  on;      

    server {      
        listen       8080;
        server_name  192.168.2.59;
                                
        #charset koi8-r;                                         

        #access_log  logs/host.access.log  main;
                                                                 
        location / {
            root   /usr/share/nginx/html;   
            index  index.html index.html;
        }
                                                                 
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #                
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
                                
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
#
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost; 

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

4. The main parts of this configuration you need to be aware of are...
```
rtmp{                                                                                                                              
        server{                                                                                                                    
                listen 1935;                                                                                                       
                chunk_size 4096;                                                                                                   
                application live{                                                                                                  
                        live on;                                                                                                   
                        record off;                                                                                                
                }                                                                                                                  
        }                                                                                                                          
}
```
and
```
server {                                                                                                                       
        listen       8080;                                                                                                         
        server_name  192.168.2.59;
```

The application string under rtmp and server "live" will be what you use to locate the rtmp server on your LAN

Under server, 192.168.2.59 will be the IP address of the machine you are using to broadcast this.

5. Once that is done, make sure you save all changes and you can do systemctl start nginx.

6. Open up OBS and go to Settings > Stream. 
    - Change the Service to Custom...
    - Enter the address rtmp://192.168.2.59/live
    - Enter a stream key "test"
    - Hit Ok/Apply
    - As for your Output Settings for OBS you will have to do a bit of experimenting depending on the installed codecs on your system.
    - Navigate back to the OBS main screen 
    - In sources give OBS a sound or media source...
    - and hopefully hitting the Start Streaming button will work :) 
    - If it does not you may have to go back to the configuration and check what you gave as an IP address or other minor syntax errors :)
    
7. With all of that done, if your OBS successfully starts streaming start up your VLC Media Player (Or Media Player of choice but that is beyond the scope of this guide) and in the toolbar navigate to Media > Open Network Stream.
    - Enter the network URL of rtmp://192.168.2.59/live/test and you should be getting a stream of whatever source you gave OBS!

Cheers~

Ubuntu Method
-------------
Copied from https://www.servermania.com/kb/articles/nginx-rtmp/

1. System Update 
    - apt-get update
    - apt-get Upgrade

2. Download required software
    - apt-get install -y git build-essential ffmpeg libpcre3 libpcre3-dev libssl-dev zlib1g-dev

4. Clone Module 
    - git clone https://github.com/sergey-dryabzhinsky/nginx-rtmp-module.git

5. Download Nginx
    - wget http://nginx.org/download/nginx-1.17.6.tar.gz
    - tar -xf nginx-1.17.6.tar.gz
    - cd nginx-1.17.6

6. Configure Nginx
    - ./configure --prefix=/usr/local/nginx --with-http_ssl_module --add-module=../nginx-rtmp-module
    - make -j 1
    - make install

7. Configure Nginx more
    - For this part you can follow the guide above or travel to https://www.servermania.com/kb/articles/nginx-rtmp/ for their configuration.

8. Start Nginx
    - /usr/local/nginx/sbin/nginx
    
9. Start streaming 
    - ffmpeg -re -i example-vid.mp4 -vcodec libx264 -vprofile baseline -g 30 -acodec aac -strict -2 -f flv rtmp://localhost/show/stream

Cheers~
