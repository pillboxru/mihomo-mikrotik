Это форк проекта [wiktorbgu/mihomo-mikrotik](https://github.com/wiktorbgu/mihomo-mikrotik).

<p align="center">
<img src="https://raw.githubusercontent.com/MetaCubeX/mihomo/Meta/docs/logo.png" width="120" alt="Mihomo Logo">
</p>

<h3 align="center">
A high-performance, multi-protocol Mihomo (Clash Meta) proxy client in Docker, optimized for MikroTik RouterOS v7.
</h3>

<p align="center">
Высокопроизводительный многопротокольный прокси-клиент Mihomo (Clash Meta) в Docker-контейнере, специально оптимизированный для MikroTik RouterOS v7.
</p>

### 📚 Documentation

<p>
  <a href="https://hub.docker.com/r/pillboxrus/mihomo-mikrotik">
    <img src="https://img.shields.io/badge/Docker%20Hub-Image%20%26%20Usage-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker Hub">
  </a>
</p>

<p>
  <a href="https://github.com/pillboxru/mihomo-mikrotik/wiki">
    <img src="https://img.shields.io/badge/GitHub-Wiki-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub Wiki">
  </a>
</p>

### Настройка DNS

Встроенный шаблон `default_config.yaml` поддерживает следующие переменные окружения контейнера:

| Переменная | По умолчанию | Назначение |
| --- | --- | --- |
| `DNS_NAMESERVER` | `https://dns.google/dns-query#SELECTOR` | Основной DNS: Google DoH через выход, выбранный в группе `SELECTOR`. |
| `DNS_DEFAULT_NAMESERVER` | `system` | Bootstrap DNS для разрешения имён самих DNS-серверов (`default-nameserver`). |
| `DNS_PROXY_SERVER_NAMESERVER` | `https://dns.google/dns-query` | Отдельный DNS для разрешения имён прокси-серверов: Google DoH без суффикса выбора выхода. |

Каждая переменная задаёт один DNS-сервер. Укажите нужное значение в ENV контейнера в RouterOS и перезапустите контейнер. Пользовательские шаблоны должны явно использовать эти переменные; готовые пользовательские конфигурации автоматически не изменяются.

#### Выбор выхода для DNS

Суффикс выхода указывается вручную и не добавляется автоматически:

| Значение `DNS_NAMESERVER` | Поведение в штатном шаблоне |
| --- | --- |
| `https://dns.google/dns-query#SELECTOR` | DoH через выход, выбранный в `SELECTOR`; значение по умолчанию. |
| `https://dns.google/dns-query` | DoH напрямую через обычную маршрутизацию контейнера, если внешняя маршрутизация не задаёт другой путь. |
| `https://dns.google/dns-query#DIRECT` | Явный выбор прямого выхода Mihomo. |
| `system` | Системный DNS контейнера. |

Имя группы регистрозависимо: используйте `#SELECTOR`, а не `#selector`. В штатном шаблоне `respect-rules` не включён: проксирование пользовательского трафика само по себе не направляет DoH без суффикса через `SELECTOR`. Шифрование DoH и передача через прокси — разные свойства.

#### Изменение поведения и ограничения

- Ранее основной DNS штатного шаблона был `system`. После обновления образа без переопределения ENV используется Google DoH через `SELECTOR`. Для возврата к прежнему основному DNS задайте `DNS_NAMESERVER=system`.
- Если выбранный выход недоступен, основной DNS тоже может перестать работать. Автоматический откат на системный или прямой DNS этими параметрами не предусмотрен.
- Если в `SELECTOR` выбран `DIRECT`, DoH идёт напрямую. Суффикс `#SELECTOR` не гарантирует использование VPN.
- `DNS_PROXY_SERVER_NAMESERVER=system` использует DNS из системных настроек контейнера. Он должен работать независимо от основного DNS Mihomo: не направляйте его обратно в резолвер, которому для ответа нужен ещё не установленный прокси.
- Разрешение имени самого DoH-сервера (`dns.google`) — отдельный bootstrap-этап, управляемый `DNS_DEFAULT_NAMESERVER` (`default-nameserver` Mihomo), по умолчанию `system`. Настройки DNS не гарантируют отсутствие всех DNS-запросов вне VPN.
- Google DoH обычно не разрешает локальные домены и внутренние DNS-зоны. Если они нужны, используйте подходящий DNS или собственный шаблон с `nameserver-policy`.
- Эти параметры не изменяют DNS-forwarder и маршрутизацию RouterOS.

Подробнее о параметрах DNS: [документация Mihomo](https://wiki.metacubex.one/en/config/dns/).
