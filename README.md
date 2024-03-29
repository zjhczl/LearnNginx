# LearnNginx
## windows
直接使用nginx安装包
## ubuntu
```
apt install nginx
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