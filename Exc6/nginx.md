
http {
   # Настройка upstream для балансировки нагрузки
   upstream backend_servers {
       server backend1.example.com;
       server backend2.example.com;
       server backend3.example.com;
   }

   # Настройка rate limiting
   limit_req_zone $binary_remote_addr zone=partner_limit:10m rate=10r/m;

   server {
       listen 80;

       location / {
           limit_req zone=partner_limit burst=5 nodelay;
           proxy_pass http://backend_servers;
           error_page 429 = /429.html;
       }

       location = /429.html {
           return 429 "Too Many Requests - Rate limit exceeded";
       }
   }
}
