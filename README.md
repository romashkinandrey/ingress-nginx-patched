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

## Использование с чартом ingress-nginx 4.15.x

```yaml
controller:
  image:
    registry: ghcr.io
    image: romashkinandrey/ingress-nginx-controller
    tag: v1.15.16
    digest: "sha256:..."   # брать из итога сборки или `docker buildx imagetools inspect`
    digestChroot: ""
```

## Проверка происхождения

```bash
gh attestation verify oci://ghcr.io/romashkinandrey/ingress-nginx-controller:v1.15.16 --owner romashkinandrey
```

Сборка повторяет рецепт CI форка; функциональная проверка образа — на стороне того, кто его использует.
