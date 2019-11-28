```
 server {
    	listen       80;
    	server_name  127.0.0.1;
    	root   /var/www/html;
    	index  index.htm index.html;
    	add_header 'Access-Control-Allow-Origin' '*';
    	add_header 'Access-Control-Allow-Credentials' 'true';
    	add_header Cache-Control private;
    	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    	add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';

​	location / {

		# 此处用于处理 H5 的 History时 重写的问题

​		if (!-e $request_filename) {
​			rewrite ^(.*) /index.html last;
​			break;
​		}
​	}

	# 代理服务端接口

​	location /login {
​		if ($request_method = 'OPTIONS') {
​			add_header Access-Control-Allow-Origin *;
​			add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,PATCH,OPTIONS;
​			return 200;
​		}
​		proxy_pass http://127.0.0.1:8000/login;   #将真正的请求代理到API 服务地址
​	}

}
```

1、nginx 配置反向代理时应该注意要修改default.conf 的配置将80端口该为其他端口

2、注意端口的开放的状态

cmake -D CMAKE_BUILD_TYPE=RELEASE \  -D CMAKE_INSTALL_PREFIX=/usr/local \  -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.0.0/modules \  -D ENABLE_NEON=ON \  -D ENABLE_VFPV3=ON \  -D BUILD_TESTS=OFF \  -D OPENCV_ENABLE_NONFREE=ON \  -D INSTALL_PYTHON_EXAMPLES=OFF \  -D BUILD_EXAMPLES=OFF ..

