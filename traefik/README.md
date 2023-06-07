# `Traefik` config files

> Этот файл - не инструкция для начинающих. Это список граблей на которые я неоднократно наступал при изучении и настройке такой прекрасной штуки как Traefik. Возможно мои заметки будут кому-нибудь полезны. А начинающим я могу посоветовать, например, [видос Технотима](https://www.youtube.com/watch?v=liV3c9m_OX8), или, еще лучше, [этот видос](https://www.youtube.com/watch?v=b83S_N1kkJM)

Файловая структура:
```konsole
traefik
│ docker-compose.yml
└ data
  │ acme.json        <- хранилище сертификатов
  │ traefik.yml      <- статический конфиг
  └ custom           <- динамические конфиги
    | header.yml     <- security headers
    | jupyter.yml 
    | portainer.yml 
    | router.yml
    | server.yml     <- для Proxmox
    | traefik.yml    <- для дашборда
```

## Если что-то не работает

Включаем более подробные логи - открываем файл `traefik/data/traefik.yml` и в самое начало дописываем:
```yaml
log:
  level: DEBUG
```

Перезапускаем контейнер:
```konsole
docker compose restart
```

И внимательно читаем его логи (можно добавить `--tail 10` для вывода 10 последних строк):
```konsole
docker logs traefik
```

## Топ-1 самая популярная причина ошибки:

Неправильно выставили, или просто забыли выставить разрешения для хранилища сертификатов
```konsole
The ACME resolver "cloudflare" is skipped from the resolvers list because:
unable to get ACME account: permissions 644 for acme.json are too open, please use 600
```

Говорит юзать `600` - не забываем:
```konsole
chmod 600 acme.json
```

## Если происходит ошибка при парсинге YAML

В своих конфигах, я активно использую [Go Templating](https://doc.traefik.io/traefik/providers/file/#go-templating). Если вам по какой-то причине это не нравится, или это вызывает у вас ошибку примерно такого вида:
```konsole
yaml: line 39: did not find expected alphabetic or numeric character
```

И вам лень разбирваться в чем проблема, то можно просто зайти на сайт https://repeatit.io (go templates playground), вставить слева мой конфиг, а справа получить чистый YAML

## Если есть проблемы с получением сертификата
Если вы, как и я, хотите себе именно Wildcard сертификаты, используя Cloudflare как провайдера для их получения, то вот вам совет:

На ремя получения сертификатов, отключите проксирование трафика в панели управления Cloudflare для всех DNS записей (чтобы вместо `Proxied` было `DNS only`). Также, может быть полезным временное включение Development Mode, который отключает кеширование

## Что еще почитать / посмотреть по теме:
- [Статья, видос и конфиги Технотима](https://docs.technotim.live/posts/traefik-portainer-ssl/)
- [Конфиги моего друга](https://github.com/nhths/server-boilerplate/tree/master/traefik) (без Wildcard, провайдер Let's Encrypt)
- [Все статьи Технотима про Traefik](https://docs.technotim.live/tags/traefik/)
- [Документация Traefik](https://doc.traefik.io/traefik/) - топ полезных ссылок:
  - [Configuration Discovery / File](https://doc.traefik.io/traefik/providers/file/)
  - [Let's Encrypt](https://doc.traefik.io/traefik/https/acme/)
  - [BasicAuth](https://doc.traefik.io/traefik/middlewares/http/basicauth/)
