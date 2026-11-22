Отчёт здесь писать надо?

Предположу, что да.

#### На 4 балла

Сделал что написано

Все команды к консоли:
```console
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> docker run -d --name ipr-web \
                                                            -p 127.0.0.1:8080:8000 \
                                                            -e GREETING='Meoooow Mrrrp !! --- container' \
                                                            ipr-web:1
cea14254f89f1df2fccd6b018dfa274c8e3298006f808d5b287c304cd5477975
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS http://127.0.0.1:8080/
{"greeting": "Meoooow Mrrrp !! --- container", "count": 0, "hostname": "cea14254f89f", "storage": "sqlite"}⏎                        
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Meoooow Mrrrp !! --- container", "count": 1, "hostname": "cea14254f89f", "storage": "sqlite"}⏎                        
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Meoooow Mrrrp !! --- container", "count": 2, "hostname": "cea14254f89f", "storage": "sqlite"}⏎                        
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Meoooow Mrrrp !! --- container", "count": 3, "hostname": "cea14254f89f", "storage": "sqlite"}⏎                        
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS http://127.0.0.1:8080/
{"greeting": "Meoooow Mrrrp !! --- container", "count": 3, "hostname": "cea14254f89f", "storage": "sqlite"}
```

Проверка логов:
```console
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> docker logs ipr-web
2026-11-18 19:56:11,751 listening port=8000 storage=sqlite
2026-11-18 19:56:54,076 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-18 19:57:13,027 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-18 19:57:16,316 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-18 19:57:17,843 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-18 19:57:23,902 http 172.17.0.1 "GET / HTTP/1.1" 200 -
```

Отдельно также они лежат в папке `/logs`... хотя в целом там тоже самое

#### На 6 баллов

Как оказалось `compose` нужно ставить отдельно. Я об этом не знал и потратил чересчур много времени на дебаг чудесной ошибки `unknown shorthand flag: 'p' in -p`.

Не знаю что надо писать, а что нет, поэтому вот весь процесс

На больше баллов я не делал, поэтому `compose.yaml`, `.env.example` и `.env` можете просто из репозитория открыть

Создаём первый контейнер
```console
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> docker compose -p ipr-web -f compose.yaml config
name: ipr-web
services:
  web:
    environment:
      DATA_FILE: /data/counter.db
      GREETING: Custom .env Hi!!! --- Compose :3
      PORT: "8000"
      STORAGE: sqlite
    networks:
      default: null
    volumes:
      - type: volume
        source: aaaaaaa
        target: /data
        volume: {}
    build:
      context: /home/soup/hse/ipr/docker/hw_3_ipr_meow
      dockerfile: Dockerfile
    ports:
      - mode: ingress
        host_ip: 127.0.0.1
        target: 8000
        published: "8080"
        protocol: tcp
networks:
  default:
    name: ipr-web_default
volumes:
  aaaaaaa:
    name: ipr-web_aaaaaaa

soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> docker compose up -d --build
[+] Building 2.0s (11/11) FINISHED                                                                                                                               
 => [internal] load local bake definitions                                                      

... много строчек

[+] up 4/4
 ✔ Image hw_3_ipr_meow-web       Built                                                                                                                       2.1s
 ✔ Network hw_3_ipr_meow_default Created                                                                                                                     0.1s
 ✔ Volume hw_3_ipr_meow_aaaaaaa  Created                                                                                                                     0.0s
 ✔ Container hw_3_ipr_meow-web-1 Started
```

```console
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS http://127.0.0.1:8080/
{"greeting": "Custom .env Hi!!! --- Compose :3", "count": 0, "hostname": "b0858a589186", "storage": "sqlite"}
```
Здесь `Custom .env Hi!!!` берётся из `.env`, а ` --- Compose :3` добавляется `сompose.yaml`-ом.

```console
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Custom .env Hi!!! --- Compose :3", "count": 1, "hostname": "b0858a589186", "storage": "sqlite"}⏎                                                   
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Custom .env Hi!!! --- Compose :3", "count": 2, "hostname": "b0858a589186", "storage": "sqlite"}⏎                                                   
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Custom .env Hi!!! --- Compose :3", "count": 3, "hostname": "b0858a589186", "storage": "sqlite"}
```
Прибавляем пару раз, вот ещё логи на всякий случай (они тоже есть в `/logs`):
```
web-1  | 2026-11-22 16:40:23,182 listening port=8000 storage=sqlite
web-1  | 2026-11-22 16:41:22,033 http 172.18.0.1 "GET / HTTP/1.1" 200 -
web-1  | 2026-11-22 16:45:53,686 http 172.18.0.1 "POST /increment HTTP/1.1" 200 -
web-1  | 2026-11-22 16:45:54,774 http 172.18.0.1 "POST /increment HTTP/1.1" 200 -
web-1  | 2026-11-22 16:45:55,462 http 172.18.0.1 "POST /increment HTTP/1.1" 200 -
web-1  | 2026-11-22 16:48:52,139 http 172.18.0.1 "GET / HTTP/1.1" 200 -
```

Удаляем первый контейнер и создаём новый:
```console
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> docker compose down
[+] down 2/2
 ✔ Container hw_3_ipr_meow-web-1 Removed                                                                                                                    10.4s
 ✔ Network hw_3_ipr_meow_default Removed                                                                                                                     0.2s
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> docker compose up -d --build
[+] Building 2.0s (11/11) FINISHED                                                           

... снова много строчек

[+] up 3/3
 ✔ Image hw_3_ipr_meow-web       Built                                                                                                                       2.1s
 ✔ Network hw_3_ipr_meow_default Created                                                                                                                     0.1s
 ✔ Container hw_3_ipr_meow-web-1 Started
```

Проверяем, `count` сохранился !!

```console
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> docker compose ps
NAME                  IMAGE               COMMAND              SERVICE   CREATED         STATUS         PORTS
hw_3_ipr_meow-web-1   hw_3_ipr_meow-web   "python -u app.py"   web       2 minutes ago   Up 2 minutes   127.0.0.1:8080->8000/tcp
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS http://127.0.0.1:8080/
{"greeting": "Custom .env Hi!!! --- Compose :3", "count": 3, "hostname": "c76b48859049", "storage": "sqlite"}⏎                                                   
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Custom .env Hi!!! --- Compose :3", "count": 4, "hostname": "c76b48859049", "storage": "sqlite"}⏎                                                   
soup@lenovo-legion-7-soup ~/h/i/d/hw_3_ipr_meow (main)> curl -fsS http://127.0.0.1:8080/
{"greeting": "Custom .env Hi!!! --- Compose :3", "count": 4, "hostname": "c76b48859049", "storage": "sqlite"}
```

Вот логи нового контейнера:
```
web-1  | 2026-11-22 16:50:59,794 listening port=8000 storage=sqlite
web-1  | 2026-11-22 16:53:43,480 http 172.18.0.1 "GET / HTTP/1.1" 200 -
web-1  | 2026-11-22 16:53:53,717 http 172.18.0.1 "POST /increment HTTP/1.1" 200 -
web-1  | 2026-11-22 16:53:56,265 http 172.18.0.1 "GET / HTTP/1.1" 200 -

```