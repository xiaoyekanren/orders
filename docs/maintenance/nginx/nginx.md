# nginx

## build in ubuntu
``` shell
apt-get install gcc g++ make libssl-dev openssl libpcre3 libpcre3-dev zlib1g zlib1g-dev libgd-dev -y
./configure --prefix=/usr/local/nginx-binary \
--user=nginx \
--group=nginx \
--with-pcre \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_stub_status_module \
--with-http_auth_request_module \
--with-http_image_filter_module \
--with-http_slice_module \
--with-mail \
--with-threads \
--with-file-aio \
--with-stream \
--with-mail_ssl_module \
--with-stream_ssl_module

make
make install
```
注1：--prefix写好的地址会写死到二进制文件里，也就是不能更改路径了，挪动路径需要指定-p -c
注2：--user, --group 同上，当然也可以在配置文件里手写，启动的时候指定参数
