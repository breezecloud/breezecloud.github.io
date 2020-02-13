---
layout: post
title:  "odoo设置nginx的反向代理及ssl"
date: 2019-06-11 08:15:00 +0800
categories: jekyll update
---

odoo设置nginx反向代理及ssl
===
本文参考：Odoo.11.Development.Cookbook.2nd.Edition一书，也可以参考https://alanhou.org/odoo12-deployment/ 及官方文档
https://www.odoo.com/documentation/12.0/setup/deploy.html<br>
假设已经安装好odoo和nginx，并且申请了CA证书（别忘了设置证书自动更新）。配置以odoo.example.com为例。关于证书的安装，可以参考我前面的文章。
1. 作为root，建立文件 /etc/nginx/sites-available/odoo-80: 
```
server { 
	listen [::]:80 ipv6only=off; 
	server_name odoo.example.com; 
	access_log /var/log/nginx/odoo80.access.log combined; 
	error_log /var/log/nginx/odoo80.error.log; 
	location / { 
		rewrite ^/(.*) https://odoo.example.com:443/$1 permanent; 
	} 
}
```

2. 建立配置文件 /etc/nginx/sites-available/odoo-443: 
```
server { 
	listen [::]:443 ipv6only=off; 
	server_name odoo.example.com; 
	ssl on; 
	ssl_certificate /etc/letsencrypt/live/odoo.example.com/fullchain.pem; 
	ssl_certificate_key /etc/letsencrypt/live/odoo.example.com/privkey.pem; 
	access_log /var/log/nginx/odoo443.access.log combined; 
	error_log /var/log/nginx/odoo443.error.log; 
	client_max_body_size 128M; 
	gzip on; 
	proxy_read_timeout 600s; 
	index index.html index.htm index.php; 
	add_header Strict-Transport-Security "max-age=31536000";
	
	proxy_set_header Host $http_host; 
	proxy_set_header X-Real-IP $remote_addr; 
	proxy_set_header X-Forward-For $proxy_add_x_forwarded_for; 
	proxy_set_header X-Forwarded-Proto https; 
	proxy_set_header X-Forwarded-Host $http_host; 
	location / { 
		proxy_pass http://localhost:8069; 
		proxy_read_timeout 6h; 
		proxy_connect_timeout 5s; 
		proxy_redirect http://$http_host/ https://$host:$server_port/; 
		add_header X-Static no; 
		proxy_buffer_size 64k; 
		proxy_buffering off; 
		proxy_buffers 4 64k; 
		proxy_busy_buffers_size 64k; 
		proxy_intercept_errors on; 
	} 
	location /longpolling/ { 
		proxy_pass http://localhost:8072; 
	} 
	location ~ /[a-zA-Z0-9_-]*/static/ { 
		proxy_pass http://localhost:8069; 
		proxy_cache_valid 200 60m; 
		proxy_buffering on; 
		expires 864000; 
	} 
} 
```

3. 建立配置文件链接 /etc/nginx/sites-enabled/: 
```
# ln -s /etc/nginx/sites-available/odoo-80 /etc/nginx/sites-enabled/odoo-80 
# ln -s /etc/nginx/sites-available/odoo-443 /etc/nginx/sites-enabled/odoo-443 
```

4. 删除默认文件/etc/nginx/sites-enabled/default: 
```
# rm /etc/nginx/sites-enabled/default 
```

5. 编辑odoo的启动配置文件
修改proxy_mode = True

6. 重启odoo和nginx，浏览http://odoo.example.com

7. 以上为Cookbook上的内容，按照此方法配置好以后还存在问题，在discuss模块无法正常进行及时聊天。后发现odoo使用长轮询机制实现聊天，应该是会打开一个8072端口。nginx已经配置了8072端口的跳转，但发现8072端口根本没有启动（无法进行反向代理，但不用反向代理情况下可以正常使用discuss）。后百度找到解决办法，把odoo的配置文件里的workers = 0调整为workers = 1，再次启动odoo服务后用lsof -i命令，可以看到8072端口已经启动。

