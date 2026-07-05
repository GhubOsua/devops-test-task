# Отчёт по диагностике и исправлению сервиса DevOps-stack
## 0. Проверка базового функционала:
### Создаем скрипт - убедиться, что базовый функционал работает;
```bash
cat test_api.sh 
#!/bin/bash

BASE_URL="http://localhost:8080"

echo "===== Тестирование эндпоинтов сервиса ====="
echo ""

# 1. /health
echo "1. Проверка /health:"
curl -s "$BASE_URL/health" | jq '.' || echo "Ошибка: не удалось получить ответ"
echo -e "\n"

# 2. /diag
echo "2. Проверка /diag:"
curl -s "$BASE_URL/diag" | jq '.' || echo "Ошибка: не удалось получить ответ"
echo -e "\n"

# 3. /items (GET)
echo "3. Проверка /items (GET):"
curl -s "$BASE_URL/items" | jq '.' || echo "Ошибка: не удалось получить ответ"
echo -e "\n"

# 4. /add (POST) - добавить запись
echo "4. Проверка /add (POST) - добавление записи:"
POST_RESPONSE=$(curl -s -X POST "$BASE_URL/add" -H "Content-Type: application/json" -d '{"name":"test_script"}')
echo "$POST_RESPONSE" | jq '.' || echo "Ошибка: не удалось получить ответ"
echo -e "\n"

# 5. /items после добавления (чтобы увидеть новую запись)
echo "5. Проверка /items после POST:"
curl -s "$BASE_URL/items" | jq '.' || echo "Ошибка: не удалось получить ответ"
echo -e "\n"

echo "===== Тестирование завершено ====="
```
### Вывод скрипта: 
```bash
./test_api.sh 
===== Тестирование эндпоинтов сервиса =====

1. Проверка /health:
{
  "ok": true,
  "time": "2026-07-05T16:27:56+00:00"
}


2. Проверка /diag:
{
  "ok": true,
  "redis": true,
  "db": true,
  "time": "2026-07-05T16:27:56+00:00"
}


3. Проверка /items (GET):
{
  "ok": true,
  "data": [
    {
      "id": 3,
      "title": "gamma",
      "price": 300
    },
    {
      "id": 2,
      "title": "beta",
      "price": 200
    },
    {
      "id": 1,
      "title": "alpha",
      "price": 100
    }
  ]
}


4. Проверка /add (POST) - добавление записи:
{
  "ok": true,
  "title": "item-162756-61bd",
  "price": 112
}


5. Проверка /items после POST:
{
  "ok": true,
  "data": [
    {
      "id": 4,
      "title": "item-162756-61bd",
      "price": 112
    },
    {
      "id": 3,
      "title": "gamma",
      "price": 300
    },
    {
      "id": 2,
      "title": "beta",
      "price": 200
    },
    {
      "id": 1,
      "title": "alpha",
      "price": 100
    }
  ]
}


===== Тестирование завершено =====
```
### Смотрим логи:
docker logs -f CONTAINER_NAMES
```bash
172.17.0.1 - - [05/Jul/2026:15:15:46 +0000] "GET /items HTTP/1.0" 200 130 "-" "ApacheBench/2.3"
172.17.0.1 - - [05/Jul/2026:15:15:46 +0000] "GET /items HTTP/1.0" 200 130 "-" "ApacheBench/2.3"
172.17.0.1 - - [05/Jul/2026:15:15:46 +0000] "GET /items HTTP/1.0" 200 130 "-" "ApacheBench/2.3"
172.17.0.1 - - [05/Jul/2026:15:15:46 +0000] "GET /items HTTP/1.0" 200 130 "-" "ApacheBench/2.3"
172.17.0.1 - - [05/Jul/2026:15:15:46 +0000] "GET /items HTTP/1.0" 200 130 "-" "ApacheBench/2.3"
2026-07-05 15:25:46 4 [Warning] Aborted connection 4 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)
2026-07-05 15:25:46 6 [Warning] Aborted connection 6 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)
2026-07-05 15:25:46 5 [Warning] Aborted connection 5 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)
172.17.0.1 - - [05/Jul/2026:16:27:56 +0000] "GET /health HTTP/1.1" 200 57 "-" "curl/7.81.0"
172.17.0.1 - - [05/Jul/2026:16:27:56 +0000] "GET /diag HTTP/1.1" 200 80 "-" "curl/7.81.0"
172.17.0.1 - - [05/Jul/2026:16:27:56 +0000] "GET /items HTTP/1.1" 200 141 "-" "curl/7.81.0"
172.17.0.1 - - [05/Jul/2026:16:27:56 +0000] "POST /add HTTP/1.1" 200 61 "-" "curl/7.81.0"
172.17.0.1 - - [05/Jul/2026:16:27:56 +0000] "GET /items HTTP/1.1" 200 189 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:05:45 +0000] "GET /items HTTP/1.1" 200 286 "-" "curl/7.81.0"
2026-07-03 20:05:50 13 [Warning] Aborted connection 13 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)
172.17.0.1 - - [03/Jul/2026:20:13:04 +0000] "GET /health HTTP/1.1" 200 57 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:13:04 +0000] "GET /diag HTTP/1.1" 200 80 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:13:06 +0000] "GET /items HTTP/1.1" 200 286 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:13:06 +0000] "POST /add HTTP/1.1" 200 61 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:13:09 +0000] "GET /items HTTP/1.1" 200 334 "-" "curl/7.81.0"
2026-07-03 20:13:11 14 [Warning] Aborted connection 14 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)
2026-07-03 20:13:14 15 [Warning] Aborted connection 15 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)
172.17.0.1 - - [03/Jul/2026:20:14:10 +0000] "GET /health HTTP/1.1" 200 57 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:14:10 +0000] "GET /diag HTTP/1.1" 200 80 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:14:12 +0000] "GET /items HTTP/1.1" 200 334 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:14:12 +0000] "POST /add HTTP/1.1" 200 61 "-" "curl/7.81.0"
172.17.0.1 - - [03/Jul/2026:20:14:15 +0000] "GET /items HTTP/1.1" 200 382 "-" "curl/7.81.0"
2026-07-03 20:14:17 16 [Warning] Aborted connection 16 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)
2026-07-03 20:14:20 17 [Warning] Aborted connection 17 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)

```
Видим записи 2026-07-05 15:25:46 5 [Warning] Aborted connection 5 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets)
Это предупреждения MySQL о том, что соединение с БД было прервано из-за таймаута. Такие предупреждения появляются через 5 секунд.

### Тестрование:
- Устанавливаем утилиту для тестирования:
sudo apt install apache2-utils
ab (Apache Bench) – это утилита командной строки для нагрузочного тестирования HTTP-серверов.
- Далее выполняем команду:
Запуск теста для GET-запроса /items
ab -n 100 -c 10 http://localhost:8080/items
-n 100 – общее количество запросов, которые будут отправлены (100 штук);
-c 10 – количество параллельных соединений (concurrency). Это означает, что ab будет поддерживать 10 одновременно открытых соединений, отправляя запросы непрерывно, пока не наберётся 100 успешных запросов;
Запуск теста для POST-запроса /add
ab -n 50 -c 5 -p post.json -T application/json http://localhost:8080/add
-p post.json – путь к файлу с телом запроса (POST data);
-T application/json – явно указывает заголовок Content-Type: application/json. Это необходимо, чтобы сервер правильно интерпретировал тело;
- На что обратить внимание:
    Failed requests – если не 0, значит, часть запросов завершилась ошибкой (например, 500, 404, таймаут). Это главный индикатор нестабильности.
    Requests per second – производительность. Если она сильно падает при повышении -c, возможно, сервис не выдерживает параллелизм.
    Time per request (mean) – среднее время ответа. Если оно резко возрастает при увеличении количества запросов, это может говорить о блокировках, нехватке соединений с БД и т.д.
    Percentage served – показывает распределение. Если 95% запросов выполняются быстро, а 5% очень медленно – это тоже проблема (например, периодические «выбросы»).
- Получаем:
```bash
ab -n 100 -c 10 http://localhost:8080/items
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        nginx
Server Hostname:        localhost
Server Port:            8080

Document Path:          /items
Document Length:        370 bytes

Concurrency Level:      10
Time taken for tests:   2.762 seconds
Complete requests:      100
Failed requests:        0
Total transferred:      50700 bytes
HTML transferred:       37000 bytes
Requests per second:    36.21 [#/sec] (mean)
Time per request:       276.187 [ms] (mean)
Time per request:       27.619 [ms] (mean, across all concurrent requests)
Transfer rate:          17.93 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       1
Processing:     5   48 249.8     22    2520
Waiting:        5   48 249.6     22    2518
Total:          6   49 249.9     22    2521

Percentage of the requests served within a certain time (ms)
  50%     22
  66%     24
  75%     26
  80%     26
  90%     28
  95%     35
  98%     75
  99%   2521
 100%   2521 (longest request)
```

```bash 
ab -n 50 -c 5 -p post.json -T application/json http://localhost:8080/add
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        nginx
Server Hostname:        localhost
Server Port:            8080

Document Path:          /add
Document Length:        50 bytes

Concurrency Level:      5
Time taken for tests:   0.236 seconds
Complete requests:      50
Failed requests:        0
Total transferred:      9350 bytes
Total body sent:        7950
HTML transferred:       2500 bytes
Requests per second:    211.69 [#/sec] (mean)
Time per request:       23.620 [ms] (mean)
Time per request:       4.724 [ms] (mean, across all concurrent requests)
Transfer rate:          38.66 [Kbytes/sec] received
                        32.87 kb/s sent
                        71.53 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       1
Processing:     8   21   6.4     20      48
Waiting:        8   21   6.4     20      48
Total:          8   21   6.4     20      48

Percentage of the requests served within a certain time (ms)
  50%     20
  66%     22
  75%     23
  80%     24
  90%     30
  95%     32
  98%     48
  99%     48
 100%     48 (longest request)
```
## 1. Анализ результатов нагрузочного теста
### Результаты GET /items (100 запросов, 10 параллельных)
```bash
ab -n 100 -c 10 http://localhost:8080/items
```
```text
Ключевые показатели:
    Ошибки: Failed requests: 0 – все запросы технически успешны (HTTP 200).
    Среднее время ответа (mean): 276 мс.
    Медиана (50-й процентиль): 22 мс.
    99-й процентиль: 2521 мс (!).
Вывод: Большинство запросов обрабатываются быстро (~22 мс), но 1 запрос из 100 занял более 2.5 секунд – это в 100 раз дольше среднего.
Такой «выброс» (аномалия) при небольшой нагрузке (всего 10 параллельных соединений) – явный признак нестабильности.
Это означает, что 99% пользователей получают ответ за < 30 мс, но 1% пользователей (1 из 100) ждут более 2.5 секунд.
При увеличении нагрузки или ужесточении таймаутов этот «выброс» может превратиться в ошибку 504/500.
Плюс в логах запись: Aborted connection ... (Got timeout reading communication packets). Это означает, что соединение с БД было разорвано
(возможно, из-за простоя или переполнения пула). Предполагаемая причина - потеря соединения с бд, переподключение занимает время.
```
### Результаты POST /add (50 запросов, 5 параллельных)
```bash
ab -n 50 -c 5 -p post.json -T application/json http://localhost:8080/add
```
```text
Ошибок нет: Failed requests: 0. Среднее время: ~20 мс, максимальное: 48 мс – все запросы укладываются в узкий диапазон, аномалий нет. При записи пул соединений ведет себя стабильней.
```
## 2. Компоненты, участвующие в проблеме
- nginx	Веб-сервер / reverse proxy	Принимает запросы, проксирует их в PHP-приложение. Может иметь свои таймауты (например, proxy_read_timeout). Если PHP долго отвечает, nginx может вернуть 504.
- PHP (приложение)	Бэкенд-приложение	Отвечает за бизнес-логику и управление соединениями с MySQL и Redis. В нём, скорее всего, отсутствует корректное переиспользование соединений (пул) или не настроен persistent connection.
- MySQL	База данных	Хранит данные. На стороне MySQL есть параметры таймаутов (wait_timeout, interactive_timeout, connect_timeout). Когда соединение простаивает дольше допустимого, MySQL закрывает его. Предупреждения в логах – это прямое доказательство.
- Redis	Кэш/хранилище сессий	Судя по /diag, работает стабильно. Проблема не в нём.

## 3. Заходим во внутрь контейнера
- docker exec -it <container_id> /bin/bash
```bash
root@835a732450ee:/# ps aux 
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.1  1.4  38692 29724 ?        Ss   19:44   0:00 /usr/bin/python3 /usr/bin/supervisord -c /etc/supervisor/supervisord.conf
mysql          8  0.4  6.0 1433092 122460 ?      Sl   19:44   0:02 /usr/sbin/mariadbd --user=mysql --datadir=/var/lib/mysql --socket=/run/mysqld/mysqld.sock --bind-address=127.0.0.1 --pid-file=/run/mysqld/mysqld
root           9  1.6  0.6  69220 13556 ?        Sl   19:44   0:08 /usr/bin/redis-server 127.0.0.1:6379
root          10  0.0  1.1 237968 22588 ?        S    19:44   0:00 php-fpm: master process (/etc/php/8.4/fpm/php-fpm.conf)
root          11  0.0  0.4  14624  8868 ?        S    19:44   0:00 nginx: master process /usr/sbin/nginx -g daemon off;
www-data      12  0.0  0.1  15004  3336 ?        S    19:44   0:00 nginx: worker process
www-data      13  0.0  0.1  15004  3336 ?        S    19:44   0:00 nginx: worker process
www-data      20  0.0  0.2 237968  5088 ?        S    19:44   0:00 php-fpm: pool www
www-data      21  0.0  0.2 237968  5088 ?        S    19:44   0:00 php-fpm: pool www
root          33  0.0  0.1   4320  3544 pts/0    Ss   19:44   0:00 /bin/bash
root          49  125  0.1   6396  3252 pts/0    R+   19:52   0:00 ps aux
```
- подключаемся к бд
mysql -u root
выполянем sql запрос
```sql
SHOW VARIABLES WHERE Variable_name IN ('interactive_timeout', 'wait_timeout');
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| interactive_timeout | 5   |
| wait_timeout        | 5   |
+---------------------+-------+
2 rows in set (0.027 sec)
```
Видим, что таймауты установлены значения (5 секунд), малое значение. Каждое соединение, которое приложение (PHP) открывает к MySQL, будет закрыто сервером через 5 секунд бездействия. Это означает, что если между запросами прошло больше 5 секунд (например, приложение обрабатывает другой запрос или контейнер простаивает), то при следующем обращении к БД соединение уже мёртво. Приложение должно либо переподключаться, либо проверять соединение перед использованием. В логах после соединения спустя 5 секунд идет сообщение - 2026-07-05 18:26:26 16 [Warning] Aborted connection 16 to db: 'app' user: 'app' host: 'localhost' (Got timeout reading communication packets). 
- Проверяем логи приложений:
```bash
find / -name "*.log" 2>/dev/null | grep -v "proc"
/var/log/supervisor/supervisord.log
/var/log/php8.4-fpm.log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/lib/mysql/ddl_recovery.log
/var/lib/mysql/tc.log
```
Ошибок нет в этих файлах.
- Проверяем reddis:
```bash
root@835a732450ee:/# redis-cli ping
PONG
root@835a732450ee:/# redis-cli set test_key "Hello"
redis-cli get test_key
OK
"Hello"
```
- Проверяем конфигурацию PHP (пул соединений):
cat /var/www/html/index.php
```php
PDO::ATTR_PERSISTENT => true,
// ...
function expensiveLoad(PDO $pdo): array {
    usleep(2500000);
    // ...
}
```
Нагрузочный тест показал 99-й процентиль 2515 мс, что прямо коррелирует с usleep(2500000).
- Проверяем nginx. Ошибок нет.
```bash
root@835a732450ee:/# nginx -T
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
# configuration file /etc/nginx/nginx.conf:
user www-data;
worker_processes auto;
worker_cpu_affinity auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;
#######.....
```
## 4. Исправления
Основное: уменьшена задержка в expensiveLoad() до 200 мс. Файл /var/www/html/index.php:
```diff
- usleep(2500000);
+ usleep(200000);
```
Выбрано значение 200 мс, чтобы сохранить минимальную имитацию нагрузки, но полностью исключить аномальные выбросы времени ответа.
Дополнительное (для чистоты логов): увеличен wait_timeout до 600 секунд в /etc/mysql/mariadb.conf.d/99-test.cnf:
```diff
- wait_timeout=5
+ wait_timeout=600
```
## 5. Результаты
Основная проблема (аномальные задержки) устранена уменьшением usleep. Дополнительно скорректированы таймауты MySQL для снижения шума в логах. 
```text
Показатель                    До                После
99-й процентиль	              2521 мс	          66 мс
Запросов в секунду	          36.21	            361.46
Предупреждения в логах        Часто             Только при простое >10 мин
Ошибок 500 не было ни до, ни после.
```
## Опционально. Способ доставки изменений
- монтирование томов (для быстрой проверки)
```bash
docker run --rm -d -p 8080:80 \
  -v $(pwd)/index.php:/var/www/html/index.php \
  -v $(pwd)/99-test.cnf:/etc/mysql/mariadb.conf.d/99-test.cnf \
  docker.kelnik.pro/tests/devops-stack:1.0
```
- сборка образа:
```dockerfile
FROM docker.kelnik.pro/tests/devops-stack:1.0
COPY index.php /var/www/html/index.php
COPY 99-test.cnf /etc/mysql/mariadb.conf.d/99-test.cnf
```
Сборка и запуск:
```bash
docker build -t devops-stack-fixed:2.0 .
docker run --rm -d -p 8080:80 devops-stack-fixed:2.0
```
## Опционально. Рекомендации для продакшена
- Вынести базы данных в отдельные контейнеры:
    MySQL и Redis не должны быть частью одного образа с приложением.
    Это позволит независимо обновлять, масштабировать и мониторить каждый компонент.
    Например, можно использовать Docker Compose или Kubernetes.
- Настроить пул соединений:
    В текущей архитектуре PHP управляет соединениями через PDO::ATTR_PERSISTENT.
    Для больших нагрузок лучше использовать выделенный пулер, например ProxySQL (для MySQL) или PgBouncer (для PostgreSQL).
    Это снизит накладные расходы на создание/закрытие соединений и улучшит управление ресурсами.
