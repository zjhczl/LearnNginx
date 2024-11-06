# LearnNginx
## windows
直接使用nginx安装包
## ubuntu
```
apt install nginx
```
## ubuntu编译安装
下载nginx包：http://nginx.org/en/download.html
解压后，编译安装
```
./configure --prefix=/usr/local/nginx --with-http_ssl_module
make
sudo make install

/usr/local/nginx/sbin/nginx
```
安装完成后，nginx可执行文件目录
```
/usr/local/nginx/sbin/nginx
```
配置文件目录
```
/usr/local/nginx/conf/nginx.conf
```
## 删除nginx
```
sudo /usr/local/nginx/sbin/nginx -s stop
sudo rm -rf /usr/local/nginx

```

## nginx结构
```
...              #全局块

events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```
1、全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
2、events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3、http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
4、server块：配置虚拟主机的相关参数，一个http中可以有多个server。
5、location块：配置请求的路由，以及各种页面的处理情况。

## 使用ubuntu默认配置文件配置php-fpm
### 安装nginx
```
apt install nginx
```
### 修改配置文件
nginx的默认配置文件在 /ect/nginx/sites-enabled/defult

在server模块下添加php-fpm配置：
```conf
location ~ \.php$ {
        #proxy_pass http://127.0.0.1:80;
	    include fastcgi_params;
		fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
		#fastcgi_pass 127.0.0.1:9000;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}
```

### 重启nginx
```
systemctl restart nginx
```
### 安装php-fpm

```
apt isntall php-fpm
```
fpm的默认配置文件在  /etc/php/7.4/fpm/pool.d/www.conf 
默认使用的sock

### 启用fpm
```
systemctl restart php7.4-fpm
```
## 使用自定义的nginx配置文件配置fpm
### fpm配置
```conf
#listen = /run/php/php7.4-fpm.sock
listen = 127.0.0.1:9000
```
修改为TCP模式
### nginx配置
```conf
location ~ \.php$ {
        #proxy_pass http://127.0.0.1:80;
	    include fastcgi_params;
		#fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}
```
### 问题
目前只能使用tcp的模式，sock模式没有配置成功。

## ubuntu使配置生效
```
sudo ln -s /etc/nginx/sites-available/my_site /etc/nginx/sites-enabled/
sudo nginx -s reload
```
## 反向代理tcp
```
stream {
    upstream backend {
        server 192.168.1.101:54321 max_fails=3 fail_timeout=30s;
        server 192.168.1.102:54321 max_fails=3 fail_timeout=30s;
        server 192.168.1.103:54321 max_fails=3 fail_timeout=30s;
    }

    server {
        listen 12345;
        proxy_pass backend;
    }
}
```
## nginx配置密码保护
要在 Nginx 中为目录设置基本的账号密码保护，您需要使用 auth_basic 指令来启用 HTTP 基本认证，并使用 auth_basic_user_file 指令指定一个包含用户名和密码的文件。以下是设置账号密码保护的步骤：

创建密码文件：首先，您需要创建一个密码文件，通常使用 htpasswd 工具来完成。这个工具可能包含在 apache2-utils 包中，因此如果您还没有安装，您可能需要先安装它。

在 Ubuntu 或 Debian 系统上，您可以使用以下命令安装 apache2-utils：
```
sudo apt-get update
sudo apt-get install apache2-utils
```
然后，使用 htpasswd 创建一个新的密码文件，并添加一个用户。以下命令会创建一个名为 .htpasswd 的文件，并添加一个名为 username 的用户：
```
sudo htpasswd -c /etc/nginx/.htpasswd username
```
您将被提示输入密码两次。-c 参数表示创建一个新文件，如果文件已经存在，它将被覆盖。如果您想添加更多用户，省略 -c 参数：
```
sudo htpasswd /etc/nginx/.htpasswd anotheruser
```
配置 Nginx：接下来，您需要更新 Nginx 的配置文件以使用这个密码文件。编辑您的 Nginx 配置文件（例如 /etc/nginx/sites-available/default），并在您想要保护的 location 块中添加 auth_basic 和 auth_basic_user_file 指令：
```
location /protected-directory/ {
    auth_basic "Restricted Content";  # 这里的文本将显示在登录提示中
    auth_basic_user_file /etc/nginx/.htpasswd;  # 指向您的密码文件

    root /usr/share/nginx/html;
    autoindex on;
}
```
确保将 /protected-directory/ 更改为您要保护的实际目录路径，并且 .htpasswd 文件的路径是正确的。

重载 Nginx 配置：保存您的配置文件，并重载 Nginx 以应用更改：
```
sudo systemctl reload nginx
```
现在，当用户尝试访问 /protected-directory/ 目录时，浏览器会提示他们输入用户名和密码。只有在输入了有效的凭据后，用户才能看到目录内容。

请注意，HTTP 基本认证本身不是非常安全，因为它没有加密用户的凭据。如果您想要更安全的解决方案，应该考虑使用 SSL/TLS 来加密您的连接，以及更高级的认证机制。

## 删除nginx
```
sudo systemctl stop nginx
sudo apt-get remove --purge nginx nginx-common
sudo apt-get autoremove
sudo rm -rf /etc/nginx
sudo apt-get clean

```

## nginx负载均衡
nginx.conf
```
stream {
    upstream backend {
        server 101.132.151.231:28002 max_fails=3 fail_timeout=30;
        server 10.58.32.11:22091 max_fails=3 fail_timeout=30;


    }
    server {
        listen 28002; # 监听的端口可以是80或其他
        proxy_pass backend; # 转发到定义的upstream
    }
}

```


