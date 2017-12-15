---
layout: post
title: nginx 配置pathInfo
date: 2015-12-16 15:59:28.000000000 +09:00
---

# 前言
毕业设计买的服务器，选用`nginx`作为服务器，用的 ThinkPHP 作为开发框架，但是`nginx`默认不支持`pathInfo`路由格式(`apache`下是默认支持的).
网上查了很多资料，但其中是不完整的，不能正确处理`pathInfo`.

首先，修改文件/etc/nginx/fastcgi_params
修改 SCRIPT_FILENAME 并添加 PATH_INFO 参数,如下：

	fastcgi_param   QUERY_STRING            $query_string;
	fastcgi_param   REQUEST_METHOD          $request_method;
	fastcgi_param   CONTENT_TYPE            $content_type;
	fastcgi_param   CONTENT_LENGTH          $content_length;
	
	fastcgi_param   SCRIPT_FILENAME         $document_root$fastcgi_script_name;
	fastcgi_param   SCRIPT_NAME             $fastcgi_script_name;
	fastcgi_param   PATH_INFO               $fastcgi_path_info;
	fastcgi_param       PATH_TRANSLATED         $document_root$fastcgi_path_info;
	fastcgi_param   REQUEST_URI             $request_uri;
	fastcgi_param   DOCUMENT_URI            $document_uri;
	fastcgi_param   DOCUMENT_ROOT           $document_root;
	fastcgi_param   SERVER_PROTOCOL         $server_protocol;
	
	fastcgi_param   GATEWAY_INTERFACE       CGI/1.1;
	fastcgi_param   SERVER_SOFTWARE         nginx/$nginx_version;
	
	fastcgi_param   REMOTE_ADDR             $remote_addr;
	fastcgi_param   REMOTE_PORT             $remote_port;
	fastcgi_param   SERVER_ADDR             $server_addr;
	fastcgi_param   SERVER_PORT             $server_port;
	fastcgi_param   SERVER_NAME             $server_name;
	
	fastcgi_param   HTTPS                   $https;
	
	# PHP only, required if PHP was built with --enable-force-cgi-redirect
	fastcgi_param   REDIRECT_STATUS         200;
	 
	
	 

然后修改 /etc/nginx/nginx.conf 文件，在最后添加

	location ~ [^/]\.php(/|$) {
	    fastcgi_split_path_info ^(.+?\.php)(/.*)$;
	    if (!-f $document_root$fastcgi_script_name) {
	        return 404;
	    }
	
	    fastcgi_pass 127.0.0.1:9000;
	    fastcgi_index index.php;
	    include fastcgi_params;
	}
	 

重启`nginx` ，执行命令 `/etc/init.d/nginx restart` 

接下来就可以使用PATH_INFO 了；

 

参考文章 [https://www.nginx.com/resources/wiki/start/topics/examples/phpfcgi/#](https://www.nginx.com/resources/wiki/start/topics/examples/phpfcgi/#)

