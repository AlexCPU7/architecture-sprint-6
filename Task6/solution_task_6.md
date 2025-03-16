# Настройка Rate Limiting

## Постановка проблемы и требования

Количество запросов от одного из партнёров всё ещё снижает производительность приложения. Это влияет на опыт использования API других партнёров. Этой проблемой тоже нужно заняться.
Одно из возможных решений — динамическое масштабирование приложения, которое вы уже внедрили. Однако ресурсы системы не бесконечные. Поэтому необходимо предусмотреть дополнительную защиту на случай, если количество запросов от партнёра снова превысит оговорённое количество.
С этим поможет паттерн Rate Limiting.

## Технологические решение

Текущая конфигурация:
```
http {
   # Настройка upstream для балансировки нагрузки
   upstream backend_servers {
       server backend1.example.com;
       server backend2.example.com;
       server backend3.example.com;
   }

   server {
       listen 80;

       location / {
           proxy_pass http://backend_servers;
       }

   }
}
```

Обновленная конфигурация:
```
http {
   # Настройка upstream для балансировки нагрузки
   upstream backend_servers {
       server backend1.example.com;
       server backend2.example.com;
       server backend3.example.com;
   }

   limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/m;

   server {
       listen 80;

       location / {
          proxy_pass http://backend_servers;

           limit_req zone=api_limit;
           limit_req_status 429;
       }

   }

}
```
[nginx.conf](nginx.conf)
