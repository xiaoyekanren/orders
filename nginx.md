# nginx

## build in ubuntu
```
apt-get install gcc g++ make libssl-dev openssl libpcre3 libpcre3-dev zlib1g zlib1g-dev libgd-dev -y
./configure --prefix=/root/src/nginx-binary --user=nginx --group=nginx --with-pcre --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module --with-http_image_filter_module --with-http_slice_module --with-mail --with-threads --with-file-aio --with-stream --with-mail_ssl_module --with-stream_ssl_module
make
make install
```
