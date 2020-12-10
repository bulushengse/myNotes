
#定义nginx运行的用户和用户组
#user  nobody;

#nginx进程数，<=CPU数
worker_processes  1;

#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
#error_log  logs/error.log;
#error_log  logs/error.log  info;

#进程保存文件
#pid        logs/nginx.pid;


events {
	#单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections  1024;
} 


http {
	#文件扩展名与文件类型映射表
    include       mime.types;
	
	#默认文件类型
    default_type  application/octet-stream;

	#日志文件输出格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

	#请求日志保存位置
    #access_log  logs/access.log  main;

	#打开发送文件
    sendfile        on;
    #tcp_nopush     on;

    #连接超时时间
    keepalive_timeout  65;

	#打开gzip压缩
    #gzip  on;
	
	#nginx负载均衡  对于大量并发请求的压力，将请求分配到不同服务器	
	#轮询（默认）
	upstream ltweb {
	  server 192.168.1.84:8992;
	  #server 192.168.1.85:8992;
	}
	

	#配置虚拟主机，基于域名、ip和端口
    server {
        listen       8800;
        server_name  localhost;

		#默认编码
        #charset koi8-r;

		#nginx访问日志放在logs/host.access.log下，并且使用main格式（还可以自定义格式）
        #access_log  logs/host.access.log  main;

		#正则表达匹配请求   
        location ^~/SSM { 
            root   html;
            index  index.html index.htm;
			#配置反向代理的ip地址和端口号
			proxy_pass http://127.0.0.1:8080;
			#转发给服务端请求，代理机host值 
			#服务端获取代理机ip和port : HttpServletRequest.getHeader("Host")
			#当Host设置为 $http_host 时，服务端获取host是真实客户端请求头的host
			#当Host设置为 $proxy_host 时，服务端获取host是上面proxy_pass配置的host
			#当Host设置为 $host 时，当真实客户端请求头的host为''，服务端获取host是上面server_name配置的值localhost
			proxy_set_header Host $http_host; 
			#客户端ip    服务端获取真实客户端ip : HttpServletRequest.getHeader("X-Real-IP")
			proxy_set_header X-Real-IP $remote_addr;
			#客户端port  服务端获取真实客户端port : HttpServletRequest.getHeader("X-Real-Port")
			proxy_set_header X-Real-Port $remote_port;
			#客户端和各级代理ip的完整ip链路   HttpServletRequest.getHeader("X-Forwarded-For")
			#当只存在一级nginx代理的时候X-Real-IP和X-Forwarded-For是一致的，
			#当存在多级代理的时候，X-Forwarded-For 就变成了如下形式 :X-Forwarded-For: 客户端ip， 一级代理ip， 二级代理ip...
			#proxy_set_header X-Forward-For $remote_addr;
			proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
			
        }
		
		location ^~/LTWeb {
            root   html;
            index  index.html index.htm;
			proxy_pass http://ltweb;
			proxy_set_header Host $proxy_host; 
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Real-Port $remote_port;
			proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
        }
		
		#错误页面及其返回地址
		#error_page  404              /404.html;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
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
