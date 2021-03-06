# nginx-cdn-proxy-cache
Configuração de Nginx Server para fazer proxy com cache de arquivos estáticos como .jpg, .png, .js, .css, etc


## Build image

docker build -f docker/Dockerfile .

## Resumo da configuração

Exemplo para proxy para um ALB da AWS: http://a1f1d5aa29c0744f68c6e0d1d2c4182f-1293383380.us-east-1.elb.amazonaws.com:80

```bash
proxy_connect_timeout 10;
proxy_read_timeout 180;
proxy_send_timeout 5;
proxy_buffer_size 16k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 96k;
proxy_temp_file_write_size 96k;
proxy_temp_path /tmp/temp_dir;
proxy_cache_path /var/www/nginx/cache levels=1:2 keys_zone=cache_one:100m inactive=1d max_size=10g;

server {
	listen 80;

	client_max_body_size 202M;

	location / {
                proxy_pass http://a1f1d5aa29c0744f68c6e0d1d2c4182f-1293383380.us-east-1.elb.amazonaws.com:80;
		proxy_set_header       Host $host;
		proxy_buffering        on;
		proxy_cache            cache_one;
		proxy_cache_valid      200  1d;
		proxy_cache_use_stale  error timeout invalid_header updating http_500 http_502 http_503 http_504;

		add_header wall  "Ola! Passei pelo Proxy EKS! Root";
                add_header X-Cache-Status $upstream_cache_status;
        }

        location /wp-admin/ {
                proxy_pass http://a1f1d5aa29c0744f68c6e0d1d2c4182f-1293383380.us-east-1.elb.amazonaws.com:80;
                proxy_set_header Host $host;
                add_header wall  "Ola! Passei pelo Proxy EKS, mas não tenho cache! Sou o wp-admin";

        }

	location ~ .*.(gif|jpg|png|css|js|jpeg|GIF|JPG|PNG|CSS|JS|JPEG)(.*) {
                proxy_pass  http://a1f1d5aa29c0744f68c6e0d1d2c4182f-1293383380.us-east-1.elb.amazonaws.com:80;
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_cache cache_one;
                proxy_cache_valid 200 302 24h;
                proxy_cache_valid 301 30d;
                proxy_cache_valid any 5m;
                expires 90d;
                add_header wall  "Ola! Passei pelo Proxy EKS e estou no Cache";
		add_header X-Cache-Status $upstream_cache_status;
        }
}

```