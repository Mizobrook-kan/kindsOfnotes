Typecho 前台链接或者后台登录出现 404?

编程 enable-php.conf 文件：

`vi /usr/local/nginx/conf/enable-php.conf`

然后改为:

```
location ~ [^/]\.php(/|$)
{
        #try_files $uri =404;
        fastcgi_pass  unix:/tmp/php-cgi.sock;
        fastcgi_index index.php;
        include fastcgi.conf;
        include pathinfo.conf;
}
```

**typecho主题配置错误记录**

### 1、开启调试模式

一开始使用的时候，安装插件或者更换什么主题都有可能导致各种各样的bug，这个时候我们最好还是开启调试模式，方便我们找到错误的原因，及时改正。

开启调试模式首先打开网站根目录下面的config.inc.php。
在代码最后加上

```
define('__TYPECHO_DEBUG__', true);
```

Let's Encrypt错误：Certbot could not find a block to include challenges in /etc/nginx/nginx.conf

2、根目录安装typecho，403错误

新主机在网站根目录下安装typecho，发现权限不对，打不开typecho安装页面。

3、nginx: [emerg] "try_files" directive is duplicate in /usr/local/nginx/conf/pathinfo.conf

enable-php.conf中已经有一个try_files指令，推荐注释掉pathinfo中的try_files指令。

4、500 Internal Privoxy Error

**Could not load template file `no-server-data` or one of its included components.**
