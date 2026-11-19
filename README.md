# ingress-nginx-patched

Неофициальная сборка контроллера [ingress-nginx](https://github.com/kubernetes/ingress-nginx) из форка
[chainguard-forks/ingress-nginx](https://github.com/chainguard-forks/ingress-nginx) (Chainguard EmeritOSS).
Апстрим заархивирован 24.03.2026, последний его выпуск — `v1.15.1`. Форк продолжает выпускать `controller-v1.15.x`
с перенесёнными исправлениями уязвимостей nginx 1.27.1, но готовых бесплатных образов не публикует.

## Что собирается

| Образ | Откуда |
|---|---|
| `ghcr.io/romashkinandrey/ingress-nginx-nginx:<тег images/nginx/TAG>-<версия>` | `images/nginx/rootfs` форка: nginx 1.27.1 с патчами из `images/nginx/rootfs/patches` |
| `ghcr.io/romashkinandrey/ingress-nginx-controller:<версия>` | контроллер форка, собранный на базовом образе выше |

Важно: файл `NGINX_BASE` в форке указывает на старый базовый образ апстрима без этих патчей, поэтому здесь
базовый образ собирается отдельно и передаётся в `BASE_IMAGE`.

Платформа — `linux/amd64`. Сборка запускается каждый день и собирает только новые теги форка; вручную — `workflow_dispatch`.

## Какие уязвимости закрыты (сборка `v1.15.16`)

Список — по патчам `images/nginx/rootfs/patches` и описаниям релизов форка на теге `controller-v1.15.16`.
Это переносы исправлений из nginx 1.29–1.31 на nginx 1.27.1; в апстримном образе `v1.15.1` их нет.
Оценка — как указана nginx.org в заголовке патча, «—» — не указана.

| CVE | Что исправлено | Оценка | В форке с |
|---|---|---|---|
| CVE-2026-123633 | use-after-free при создании потока HTTP/2 (при проксировании в upstream) | critical | 1.15.15 |
| CVE-2026-123007 | переполнение окна управления потоком HTTP/2, рост памяти | critical | 1.15.15 |
| CVE-2026-123634 | use-after-free в резолвере при повторном разрешении имени (серия «NGINX2Root») | — | 1.15.16 |
| CVE-2026-121810 | use-after-free в HTTP/3 (QUIC) при исчерпании `worker_connections` | — | 1.15.16 |
| CVE-2026-118899 | бесконечный цикл воркера при чтении тела запроса по TLS | — | 1.15.16 |
| CVE-2026-125283 | переполнение буфера воркера при `try_files` вместе с `alias` | — | 1.15.16 |
| CVE-2026-120967 | клиентские сертификаты в `stream` проверялись по настройкам сервера по умолчанию | — | 1.15.16 |
| CVE-2026-123640 | `ssl_preread` иначе, чем TLS, разбирал некорректный ClientHello | — | 1.15.16 |
| CVE-2026-9256 | переполнение кучи в модуле rewrite | — | 1.15.6 |
| CVE-2026-42945 | переполнение кучи в модуле rewrite | — | 1.15.5 |
| CVE-2026-49975 | отказ в обслуживании HTTP/2 без авторизации; добавлена директива `max_headers` | — | 1.15.7 |
| CVE-2026-42055 | нет ограничения длины заголовков от upstream по HTTP/2 и gRPC | — | 1.15.8 |
| CVE-2026-48142 | выход за границу буфера в модуле charset | — | 1.15.8 |
| CVE-2026-42533 | переполнение буфера при вычислении скриптов (регулярные выражения) | — | 1.15.10 |
| CVE-2026-93479 | переполнение буфера при вычислении скриптов (резерв памяти) | — | 1.15.12 |
| CVE-2026-60005 | раскрытие памяти воркера из-за устаревших захватов регулярных выражений | — | 1.15.12 |
| CVE-2026-97205 | use-after-free при асинхронном файловом вводе-выводе (AIO) | medium | 1.15.12 |
| CVE-2026-56434 | use-after-free из-за двойного завершения подзапроса | medium | 1.15.12 |
| CVE-2026-40701 | use-after-free в резолвере OCSP (`ssl_stapling`) | medium | 1.15.12 |
| CVE-2026-97629 | подделка TLS session ticket: ключи синхронизировались не все | medium | 1.15.12 |
| CVE-2026-1642 | подмешивание открытого текста от upstream по TLS | medium | 1.15.12 |
| CVE-2026-95101 | подмешивание тела запроса в HTTP/3 | medium | 1.15.12 |
| CVE-2026-40460 | подмена адреса при миграции соединения QUIC (HTTP/3) | medium | 1.15.12 |
| CVE-2026-42934 | чтение за границей буфера в модуле charset (`recode_from_utf8`) | low | 1.15.12 |

Кроме nginx, форк обновил Go до 1.26.2 (исправления стандартной библиотеки), `gomarkdown` (CVE-2026-40890, GHSA-cv23-7vc5-jfh7),
`golang.org/x/net`, `grpc`, OpenTelemetry, а в коде контроллера — проверку номеров портов и экранирование значений из Ingress в логах.
Уже были в апстриме `v1.15.1`: CVE-2025-23419 (nginx) и CVE-2026-4342 (контроллер).

Для каждой новой сборки список CVE-патчей выводится в итог запуска GitHub Actions.

## Опубликованные образы

| Версия | Образ | Digest |
|---|---|---|
| v1.15.16 | `ghcr.io/romashkinandrey/ingress-nginx-controller:v1.15.16` | `sha256:8cd9bae7f5c0788a7103a591662cdfe0b6f91cac4ad106ae0bd532ff75e3fddc` |
| v1.15.16 (базовый) | `ghcr.io/romashkinandrey/ingress-nginx-nginx:v2.2.26-v1.15.16` | `sha256:5c80ee37732ce16c8b3c52c3e2a1c2055dbe1e260b75894c68a84e68e18d9081` |

Только `linux/amd64`. Ставить по digest, а не по тегу: тег можно перезаписать, digest — нет.
Базовый образ нужен только для сборки, в кластер ставится образ контроллера.

## Как поставить

Чарт — штатный `ingress-nginx` 4.15.x из `https://kubernetes.github.io/ingress-nginx`, меняется только образ контроллера.

### 1. Проверить образ перед выкладкой

```bash
# digest тега совпадает с таблицей выше
crane digest ghcr.io/romashkinandrey/ingress-nginx-controller:v1.15.16
# sha256:8cd9bae7f5c0788a7103a591662cdfe0b6f91cac4ad106ae0bd532ff75e3fddc

# образ собран workflow этого репозитория (подпись GitHub)
gh attestation verify \
  oci://ghcr.io/romashkinandrey/ingress-nginx-controller@sha256:8cd9bae7f5c0788a7103a591662cdfe0b6f91cac4ad106ae0bd532ff75e3fddc \
  --owner romashkinandrey

# версия и признак исправлений внутри
docker run --rm --entrypoint /nginx-ingress-controller \
  ghcr.io/romashkinandrey/ingress-nginx-controller@sha256:8cd9bae7f5c0788a7103a591662cdfe0b6f91cac4ad106ae0bd532ff75e3fddc --version
# Release: v1.15.16, nginx version: nginx/1.27.1
```

### 2. Values для helm

```yaml
controller:
  image:
    registry: ghcr.io
    image: romashkinandrey/ingress-nginx-controller
    tag: v1.15.16
    digest: "sha256:8cd9bae7f5c0788a7103a591662cdfe0b6f91cac4ad106ae0bd532ff75e3fddc"
    digestChroot: ""        # chroot-вариант не собирается; оставить пустым
```

`digest` задавать обязательно: иначе чарт подставит свой digest апстримного образа, и pull не пройдёт.
Остальные values не меняются; образ webhook (`kube-webhook-certgen`) остаётся штатным.

```bash
helm upgrade ingress-nginx ingress-nginx/ingress-nginx --version 4.15.1 -n ingress-nginx \
  --reuse-values -f image-values.yaml --wait --timeout 25m
```

То же в helmfile — второй файл values в релизе:

```yaml
releases:
  - name: ingress-nginx
    chart: ingress-nginx/ingress-nginx
    version: 4.15.1
    values:
      - values/default.yaml
      - values/image-patched.yaml   # блок controller.image выше
```

### 3. На что смотреть при выкладке

- Контроллер в виде DaemonSet меняется по одному узлу; при долгих WebSocket старый под ждёт закрытия соединений
  (`worker-shutdown-timeout` 240 с), поэтому на узел уходит 4–5 минут. Таймаут helm 10 минут для 5 узлов мал — ставить 25 минут.
- После замены: `kubectl -n <ns> get pods -o jsonpath='{..image}'` — везде `@sha256:8cd9bae7…`;
  в логах контроллера нет `[emerg]`; `nginx_ingress_controller_config_last_reload_successful` = 1.

### 4. Откат

```bash
helm rollback ingress-nginx <предыдущая ревизия> -n ingress-nginx --wait --timeout 25m
```

или вернуть штатный образ апстрима: `registry.k8s.io/ingress-nginx/controller:v1.15.1@sha256:594ceea76b01c592858f803f9ff4d2cb40542cae2060410b2c95f75907d659e1`
(в values — `registry: registry.k8s.io`, `image: ingress-nginx/controller`, `tag: v1.15.1`, этот digest).

## Новые версии

Workflow каждый день проверяет теги `controller-v*` форка и собирает только новые. Digest и список CVE-патчей
каждой сборки — в итоге запуска на вкладке Actions; таблица «Опубликованные образы» выше обновляется вручную.

Сборка повторяет рецепт CI форка; функциональная проверка образа на своей конфигурации — на стороне того, кто его использует.
