## Https 配置

假设域名为: ssl.suyo.tech

cd ~/boilerplate-docker/

1. 修改nginx配置
    ```
        location ^~ /.well-known {
            allow all;
            root  /var/www/html/ssl/;
        }
    ```

2. 启动letsencrypt

    docker run -it --rm \
      -v $(pwd)/configs/certs:/etc/letsencrypt \
      -v $(pwd)/html/ssl:/data/letsencrypt \
      docker.io/certbot/certbot \
      certonly \
      --email whenjonny@gmail.com \
      --webroot --webroot-path=/data/letsencrypt \
      --agree-tos \
      -d ssl.suyo.tech

3. 更新letsencrypt

    docker run -t --rm \
      -v $(pwd)/certs:/etc/letsencrypt \
      -v $(pwd)/html/ssl:/data/letsencrypt \
      docker.io/certbot/certbot \
      renew \
      --webroot --webroot-path=/data/letsencrypt

    docker restart nginx

4. crontab Job
    0 0 */15 * * docker run -t --rm -v $(pwd)/certs:/etc/letsencrypt -v $(pwd)/html/ssl:/data/letsencrypt -v /var/log/letsencrypt:/var/log/letsencrypt deliverous/certbot renew --webroot --webroot-path=/data/letsencrypt && docker restart nginx >/dev/null 2>&1


5. SSL Nginx 配置更改
    server {
        listen      443           ssl http2;
        listen [::]:443           ssl http2;
        server_name               ssl.suyo.tech;

        ssl                       on;

        add_header                Strict-Transport-Security "max-age=31536000" always;

        ssl_session_cache         shared:SSL:20m;
        ssl_session_timeout       10m;

        ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers               "ECDH+AESGCM:ECDH+AES256:ECDH+AES128:!ADH:!AECDH:!MD5;";

        ssl_stapling              on;
        ssl_stapling_verify       on;
        resolver                  8.8.8.8 8.8.4.4;

        ssl_certificate           /etc/letsencrypt/live/ssl.suyo.tech/fullchain.pem;
        ssl_certificate_key       /etc/letsencrypt/live/ssl.suyo.tech/privkey.pem;
        ssl_trusted_certificate   /etc/letsencrypt/live/ssl.suyo.tech/chain.pem;

        access_log                /dev/stdout;
        error_log                 /dev/stderr info;

        # other configs
    }
