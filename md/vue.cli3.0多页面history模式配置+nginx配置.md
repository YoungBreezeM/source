### 一、vue.config.js 配置

首先我们在项目下新建vue.config.js

配置如下

```js
/**
 * *@2019-11-07
 * *@author yqf
 * *@describe vue-cli 3.x配置文件
 */
// const path = require('path');
// const vConsolePlugin = require('vconsole-webpack-plugin'); // 引入 移动端模拟开发者工具 插件 （另：https://github.com/liriliri/eruda）
const CompressionPlugin = require('compression-webpack-plugin'); //Gzip
// const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin; //Webpack包文件分析器
// const baseUrl = process.env.NODE_ENV === "production" ? "/static/" : "/"; //font scss资源路径 不同环境切换控制
let devServer = require("./config/devServer.js");
let cssConfig = require("./config/cssConfig.js");
module.exports = {
  //基本路径
  //baseUrl: './',//vue-cli3.3以下版本使用
  publicPath: '/',//vue-cli3.3+新版本使用
  //输出文件目录
  outputDir: './dist',
  // eslint-loader 是否在保存的时候检查
  lintOnSave: true,
  //放置生成的静态资源 (js、css、img、fonts) 的 (相对于 outputDir 的) 目录。
  assetsDir: 'static',
  //build时候开启gzip
  // eslint-disable-next-line no-unused-vars
  configureWebpack:config=>{
    if(process.env.NODE_ENV === 'production'){
      return{
        plugins: [
          new CompressionPlugin({
            test:/\.js$|\.html$|.\css/, //匹配文件名
            threshold: 10240,//对超过10k的数据压缩
            deleteOriginalAssets: false //不删除源文件
          })
        ]
      }
    }
  },
  //以多页模式构建应用程序。
  pages: {
    index: {
      // page 的入口
      entry: './src/pages/index/main.js',
      // 模板来源
      template: 'public/index.html',
      // 在 dist/index.html 的输出
      filename: 'index.html',
      // 当使用 title 选项时，
      // template 中的 title 标签需要是 <title><%= htmlWebpackPlugin.options.title %></title>
      title: 'index page',
      chunks: ['chunk-vendors', 'chunk-common', 'index']
    },
    whole: {
      // page 的入口
      entry: './src/pages/whole/whole.js',
      // 模板来源
      template: 'public/whole.html',
      // 在 dist/index.html 的输出
      filename: 'whole.html',
      // 当使用 title 选项时，
      // template 中的 title 标签需要是 <title><%= htmlWebpackPlugin.options.title %></title>
      title: 'whole page',
      chunks: ['chunk-vendors', 'chunk-common', 'whole']
    }
  },
  //是否使用包含运行时编译器的 Vue 构建版本
  runtimeCompiler: false,
  //是否为 Babel 或 TypeScript 使用 thread-loader。该选项在系统的 CPU 有多于一个内核时自动启用，仅作用于生产构建，在适当的时候开启几个子进程去并发的执行压缩
  parallel: require('os').cpus().length > 1,
  //生产环境是否生成 sourceMap 文件，一般情况不建议打开
  productionSourceMap: false,
  //调整 webpack 配置 https://cli.vuejs.org/zh/guide/webpack.html#%E7%AE%80%E5%8D%95%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%B9%E5%BC%8F
  css: cssConfig,
  // webpack-dev-server 相关配置 https://webpack.js.org/configuration/dev-server/
  devServer: devServer,
  // 第三方插件配置 https://www.npmjs.com/package/vue-cli-plugin-style-resources-loader
  // pluginOptions:""
};
```

配置devserver 环境

```js
const devServer = {
  // host: 'localhost',
  host: "127.0.0.1",
  port: 8083, // 端口号
  https: false, // https:{type:Boolean}
  open: true, //配置自动启动浏览器  http://172.11.11.22:8888/rest/XX/
  hotOnly: true, // 热更新
  // proxy: 'http://localhost:8000',   // 配置跨域处理,只有一个代理
  // http://127.0.0.1:8080/ElectricityPriject/rest/JsonData/allcitys
  proxy: {
    '/ElectricityPriject/rest/JsonData/FourModual': {
      target: 'http://127.0.0.1:8080',   // 需要请求的地址
      changeOrigin: true,  // 是否跨域
      pathRewrite: {// 重写target中的请求地址，也就是说，在请求的时候，url用'/login'增加为'http://127.0.0.1:8000/login'
        '^/ElectricityPriject/rest/JsonData/FourModual': '/ElectricityPriject/rest/JsonData/FourModual'
      }
    },
  },
  historyApiFallback: {
    rewrites: [//页面重写
      { from: /\/index/, to: '/index.html' },
      { from: /\/toLogin/, to: '/login.html' },
      { from: /\/whole/, to: '/whole.html' }
    ]
  }
}
module.exports = devServer;
```

这里把长的配置对象单独分离出来不会让整个项目看起来很臃肿，这样就配置好了vue 多页面配置



### 二、nginx配置

nginx 的安装就不详细的介绍了

首先找到nginx.conf 配置文件，配置如下

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	# 开启 资源压缩
	##

	gzip on;

	 gzip_vary on;
	 gzip_proxied any;
	 gzip_comp_level 5;
	 gzip_buffers 16 8k;
	 gzip_http_version 1.1;
	 gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
     
	 #站点服务配置
    server {
    	listen       80;
    	server_name  127.0.0.1;
    	add_header 'Access-Control-Allow-Origin' '*';
    	add_header 'Access-Control-Allow-Credentials' 'true';
    	add_header Cache-Control private;
    	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    	add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';

    	# 反向代理服务端接口
    	location /ElectricityPriject/rest/JsonData/FourModual {
			#设置跨域请求
    		if ($request_method = 'OPTIONS') {
    			add_header Access-Control-Allow-Origin *;
    			add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,PATCH,OPTIONS;
    			return 200;
    		}
			#将真正的请求代理到API 服务地址
    		proxy_pass http://127.0.0.1:8080/ElectricityPriject/rest/JsonData/FourModual;   
    	}
		######
		# root 代表单前 路由所志向的项目资源路径
		# 说明 每个location 代表着一个单独路由
		#  index 代表这个路由下的的根结点（也就是多页面中的页面-html）
		#####
        #服务首页
		location / {
			root   /var/www/html;
			index   index.html;
			# 在根目录里查找资源，找到后加载资源，找不到重写路径
			try_files $uri $uri/ @rewrites;
    	}
		#数据展示页面
		location /whole {
          root   /var/www/html;
          index whole.html;
          try_files $uri $uri/ /whole.html;
       }

		location @rewrites {
             rewrite ^(.+)$ /index.html last;
        }
    }

}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
# 
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}

```

配置完之后，通过  npm run build 生成的打包文件放在nginx 配置的文件根目录下

让后通过 以下命令重启nginx

```shell
sudo systemctl reload nginx
```

这样子配置就算完成了