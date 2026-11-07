<p align="center">
<img src="https://raw.githubusercontent.com/MetaCubeX/mihomo/Meta/docs/logo.png" width="120" alt="Mihomo Logo">
</p>

<h3 align="center">
A high-performance, multi-protocol Mihomo (Clash Meta) proxy client in Docker, optimized for MikroTik RouterOS v7.
</h3>

<p align="center">
Высокопроизводительный многопротокольный прокси-клиент Mihomo (Clash Meta) в Docker-контейнере, специально оптимизированный для MikroTik RouterOS v7.
</p>

<p align="center">
<a href="https://hub.docker.com/r/wiktorbgu/mihomo-mikrotik">
<img src="https://img.shields.io/docker/pulls/wiktorbgu/mihomo-mikrotik?style=flat-square&logo=docker&logoColor=white" alt="Docker Pulls">
</a>
<a href="https://hub.docker.com/r/wiktorbgu/mihomo-mikrotik">
<img src="https://img.shields.io/docker/stars/wiktorbgu/mihomo-mikrotik?style=flat-square&logo=docker&logoColor=white" alt="Docker Stars">
</a>
<a href="https://github.com/wiktorbgu/mihomo-mikrotik">
<img src="https://img.shields.io/github/stars/wiktorbgu/mihomo-mikrotik?style=flat-square&logo=github" alt="GitHub Stars">
</a>
<a href="https://t.me/it_network_people">
<img src="https://img.shields.io/badge/Telegram-Community-purple?style=flat-square&logo=telegram&logoColor=white" alt="Telegram Community">
</a>
<a href="https://telegra.ph/Youre-making-the-world-a-better-place-11-24">
<img src="https://img.shields.io/badge/Donate-Support%20the%20Developer-green?style=flat-square&logo=heart&logoColor=white" alt="Donate">
</a>
</p>

### 📚 Documentation

<p>
  <a href="https://hub.docker.com/r/wiktorbgu/mihomo-mikrotik">
    <img src="https://img.shields.io/badge/Docker%20Hub-Image%20%26%20Usage-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker Hub">
  </a>
</p>

<p>
  <a href="https://github.com/wiktorbgu/mihomo-mikrotik/wiki">
    <img src="https://img.shields.io/badge/GitHub-Wiki-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub Wiki">
  </a>
</p>

### Настройка DNS

Встроенный шаблон `default_config.yaml` поддерживает следующие переменные окружения контейнера:

| Переменная | По умолчанию | Назначение |
| --- | --- | --- |
| `DNS_NAMESERVER` | `https://dns.google/dns-query#SELECTOR` | Основной DNS: Google DoH через выход, выбранный в группе `SELECTOR`. |
| `DNS_PROXY_SERVER_NAMESERVER` | `system` | Отдельный DNS для разрешения имён прокси-серверов, необходимый для установления соединения с ними. |

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
- Разрешение имени самого DoH-сервера (`dns.google`) — отдельный bootstrap-этап, управляемый `default-nameserver` Mihomo. Эти два ENV не меняют его настройки и не гарантируют отсутствие всех DNS-запросов вне VPN.
- Google DoH обычно не разрешает локальные домены и внутренние DNS-зоны. Если они нужны, используйте подходящий DNS или собственный шаблон с `nameserver-policy`.
- Эти параметры не изменяют DNS-forwarder и маршрутизацию RouterOS.

Подробнее о параметрах DNS: [документация Mihomo](https://wiki.metacubex.one/en/config/dns/).
