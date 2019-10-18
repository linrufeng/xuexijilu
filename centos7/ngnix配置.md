#Nginx用户及组：用户 组。window下不指定
#user  nobody;

#工作进程：数目。根据硬件调整，通常等于CPU数量或者2倍于CPU。
worker_processes  1;


#错误日志：存放路径。
#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
#error_log  logs/SSMerror.log;
#error_log  logs/SSMerror.log  notice;
#error_log  logs/SSMerror.log  info;


#pid（进程标识符）：存放路径
#pid        logs/nginx.pid;


#指定进程可以打开的最大描述符：数目
#工作模式与连接数上限
#这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
#现在在linux 2.6内核下开启文件打开数为65535，worker_rlimit_nofile就相应应该填写65535。
#这是因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
#worker_rlimit_nofile 204800;

events {

    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型
    #是Linux 2.6以上版本内核中的高性能网络I/O模型，linux建议epoll，如果跑在FreeBSD上面，就用kqueue模型。
    #补充说明：
    #与apache相类，nginx针对不同的操作系统，有不同的事件模型
    #A）标准事件模型
    #Select、poll属于标准事件模型，如果当前系统不存在更有效的方法，nginx会选择select或poll
    #B）高效事件模型
    #Kqueue：使用于FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X.使用双处理器的MacOS X系统使用kqueue可能会造成内核崩溃。
    #Epoll：使用于Linux内核2.6版本及以后的系统。
    #/dev/poll：使用于Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+。
    #Eventport：使用于Solaris 10。 为了防止出现内核崩溃的问题， 有必要安装安全补丁。
    #use epoll;
 
    
    #设置一个worker能够同时打开的最大连接数，该值最大为worker_rlimit_nofile的值
    #单个进程最大连接数（最大连接数=连接数*进程数）
    #根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为。
    #在nginx作为http服务器的时候，最大连接数为worker_processes *  worker_connctions
    #在nginx作为反向代理服务器的时候，最大连接数为worker_processes * worker_connections / 2
    worker_connections  1024;

    
    
    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。
    #分页大小可以用命令getconf PAGESIZE 取得。
    #[root@web001 ~]# getconf PAGESIZE
    #4096
    #但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。
    #client_header_buffer_size 4k;
    
    
    
    #这个将为打开文件指定缓存，默认是没有启用的，
    #max指定缓存数量，建议和打开文件数一致，
    #inactive是指经过多长时间文件没被请求后删除缓存。
    #open_file_cache max=65535 inactive=60s;
    
    
    #这个是指多长时间检查一次缓存的有效信息。
    #语法:open_file_cache_valid time 默认值:open_file_cache_valid 60 使用字段:http, server, location 这个指令指定了何时需要检查open_file_cache中缓存项目的有效信息.
    #open_file_cache_valid 80s;
    
    
    
    #open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。
    #语法:open_file_cache_min_uses number 默认值:open_file_cache_min_uses 1 使用字段:http, server, location  这个指令指定了在open_file_cache指令无效的参数中一定的时间范围内可以使用的最小文件数,如果使用更大的值,文件描述符在cache中总是打开状态.
    #open_file_cache_min_uses 1;
    
    
    #语法:open_file_cache_errors on | off 默认值:open_file_cache_errors off 使用字段:http, server, location 这个指令指定是否在搜索一个文件是记录cache错误.
    #open_file_cache_errors on;
}


http {

    #文件扩展名与文件类型映射表, 类型由mime.type文件定义
    include       mime.types;
    
    #默认文件类型
    default_type  application/octet-stream;

    
    #默认编码
    #charset utf-8;
    
    
    #服务器名字的hash表大小
    #保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket #size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket #size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket #size的提示，那么首要的是增大前一个参数的大小.
    #server_names_hash_bucket_size 128;

    
    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    #client_header_buffer_size 32k;

    
    #客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
    #large_client_header_buffers 4 64k;

    
    
    #设定通过nginx上传文件的大小
    client_max_body_size 8m;
    
    
    
    #缓冲区代理缓冲用户端请求的最大字节数，
    #如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
    #无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
    client_body_buffer_size 512k;
    
    
    
    #日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';
                        log_format log404 '$status [$time_local] $remote_addr $host$request_uri $sent_http_location';
                        

    access_log  logs/MainAccess.log  main;

    
    
    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为on，
    #如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    #sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile        on;
    
    
    
    #开启目录列表访问，合适下载服务器，默认关闭。
    #autoindex on;
    
    
    
    #此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    #tcp_nopush     on;

    
    
    #长连接超时时间，单位是秒
    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    
    
    #####################################################################################################
    proxy_temp_path tempFile;
    ## keys_zone 设置内存缓存空间名称和大小为 20MB
    ## inactive 设置缓存 1天(1d  分钟是 1m), 没有被访问的内容自动清除 (这里指的是删除文件)
    ## max_size 硬盘缓存空间大小为 100M, (G都可使用)。
    ## use_temp_path 如果为 on，则内容首先被写入临时文件 (proxy_temp_path)，然后重命名到proxy_cache_path指定的目录；如果设置为off，则内容直接被写入到 proxy_cache_path 指定的目录，如果需要cache建议off，则该特性是1.7.10提供的。
    ## proxy_temp_path 和 proxy_cache_path指定的路径必须在同一分区
    proxy_cache_path cacheFile levels=1:2 keys_zone=cache_zone:20m inactive=1d max_size=100m use_temp_path=off;


    
    
    #负载均衡配置
    upstream backend {
     
        #upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
        server 127.0.0.1:8080 weight=10 max_fails=1 fail_timeout=1s;
        #server 192.168.80.122:80 weight=2;
        #server 192.168.80.123:80 weight=3;

        #nginx的upstream目前支持4种方式的分配
        #1、轮询（默认）
        #每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
        #2、weight
        #指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
        #例如：
        #upstream bakend {
        #    server 192.168.0.14 weight=10;
        #    server 192.168.0.15 weight=10;
        #}
        #2、ip_hash
        #每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
        #例如：
        #upstream bakend {
        #    ip_hash;
        #    server 192.168.0.14:88;
        #    server 192.168.0.15:80;
        #}
        #3、fair（第三方）
        #按后端服务器的响应时间来分配请求，响应时间短的优先分配。
        #upstream backend {
        #    server server1;
        #    server server2;
        #    fair;
        #}
        #4、url_hash（第三方）
        #按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
        #例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
        #upstream backend {
        #    server squid1:3128;
        #    server squid2:3128;
        #    hash $request_uri;
        #    hash_method crc32;
        #}

        #tips:
        #upstream bakend{#定义负载均衡设备的Ip及设备状态}{
        #    ip_hash;
        #    server 127.0.0.1:9090 down;
        #    server 127.0.0.1:8080 weight=2;
        #    server 127.0.0.1:6060;
        #    server 127.0.0.1:7070 backup;
        #}
        #在需要使用负载均衡的server中增加 proxy_pass http://bakend/;

        #每个设备的状态设置为:
        #1.down表示单前的server暂时不参与负载
        #2.weight为weight越大，负载的权重就越大。
        #3.max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
        #4.fail_timeout:max_fails次失败后，暂停的时间。
        #5.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

        #nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
        #client_body_in_file_only设置为On 可以讲client post过来的数据记录到文件中用来做debug
        #client_body_temp_path设置记录文件的目录 可以设置最多3层目录
        #location对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
    }
    
    server {
    
        #监听端口
        listen       80;
        
        #域名可以有多个，用空格隔开
        server_name  localhost;

        #charset koi8-r;

        
        access_log  logs/host.access.log  main;

        
        location / {
        
            proxy_pass        http://backend;  
            #proxy_redirect off;
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header  Host  $host;
            proxy_set_header  X-Real-IP  $remote_addr;  
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
            
            
            ### 以下是缓存配置, 注意托底在这里可以实现
            ### 1、这里所谓的托底其实是跟设置的缓存时间有关。
            ### 2、通过 http 地址访问后台服务， 根据返回值生成缓存文件。
            ### 3、在 proxy_cache_valid 时间内会一直从缓存中读取, 如果 tomcat 挂掉也会读取, 这时候如果时间过期了, 就要看 proxy_cache_path 中的设置的删除文件, 如果还存在依然可以读取.
            ### 4、根据第三条, 可以在缓存时间调短, 然后把文件删除时间调长, 托底功能即可完成.
            
            ## 使用缓存, cache_zone 是上面 http 块配置的 (proxy_cache_path)
            proxy_cache cache_zone;
            
            ## 生成文件的 key 方式, 注意 args 这里, 如果不指定那么换了参数则还是同一个文件, 因为等于没有参数的路径
            proxy_cache_key "$scheme$request_method$host$request_uri$is_args$args";
            
            ## 缓存所有 1d = 1天过期,  1m = 一分钟过期
            ## proxy_cache_valid 200 302 10m;    #10分钟过期
            ## proxy_cache_valid 404 1m;
            proxy_cache_valid any 1m;
            
            ## 设置绕过缓存的请求url，即url中包含该配置的值，则该请求不从缓存中获取数据，非必须配置, 而这里则是  msg=XX  有 msg 的参数则不走缓存
            #proxy_cache_bypass $arg_msg;
            
            
            ## 说明 该指令设置与代理服务器的读超时时间。它决定了nginx会等待多长时间来获得请求的响应。这个时间不是获得整个response的时间，而是两次reading操作的时间。
            proxy_read_timeout 2s;
            
            ## 说明 该指令设置与upstream server的连接超时时间，有必要记住，这个超时不能超过75秒。这个不是等待后端返回页面的时间
            proxy_connect_timeout 2s;
            
            ## 此处是托底配置，旧的总比出错强，当nginx请求后台服务器报错的时候，如果返回配置的错误响应码，nginx则直接取缓存文件中的旧数据返回给用户，托底使用必选配置
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504 http_404;
            
        }
        
 
        
        #静态文件，nginx自己处理，不去backend请求tomcat
        location  ~* /download/ {  
            root /apps/oa/fs;  

        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        
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