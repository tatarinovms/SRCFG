# SRCFG — Shadowrocket Configuration Generator

## О проекте

Репозиторий конфигураций для Shadowrocket (iOS).

### Структура

- `*.conf` — файлы конфигураций Shadowrocket (один файл = одна конфигурация)
- `lists/` — наборы правил (rule-set), подключаемые через RULE-SET

### Конвенции проекта

- Базовый GeoIP: `GEOIP,RU,DIRECT` (Россия, не Китай)
- Стратегия по умолчанию: `FINAL,DIRECT`
- DNS: xbox-dns.ru, Quad9, Google DNS
- Raw-ссылки: `https://raw.githubusercontent.com/tatarinovms/SRCFG/master/`

---

## Формат конфигурации Shadowrocket

Конфигурация состоит из секций: `[General]`, `[Proxy]`, `[Proxy Group]`, `[Rule]`, `[Host]`, `[URL Rewrite]`, `[MITM]`.

Порядок важен — при редактировании соблюдайте существующий порядок секций.

---

### [General] — общие параметры

# Быстрый старт Shadowrocket:
# 1. Главная > Добавить узел.
# 2. Настройки > Метод проверки задержки, выберите CONNECT.
# 3. Главная > Проверка连通性, выберите доступный узел для подключения.
# При первом запуске будет предложено установить VPN-профиль. Нажмите «Хорошо» и «Разрешить».

# Если добавление/обновление подписки не удалось:
# 1. Выберите доступный узел на главной, затем Главная > Глобальный маршрут > Прокси, после чего добавьте/обновите подписку.
# 2. Переключите сетевое подключение (например, отключите VPN, смените сотовые данные на Wi-Fi или Wi-Fi на сотовые), затем добавьте/обновите подписку.
# 3. Проверьте, не устарела ли подписка, и получите корректный адрес.

# Включение HTTPS-расшифровки в Shadowrocket:
# 1. Нажмите ⓘ рядом с «Конфигурация» > HTTPS-расшифровка > Сертификат > Создать новый CA-сертификат > Установить сертификат.
# 2. Системные настройки > Загруженный профиль > Установить.
# 3. Системные настройки > Основные > О телефоне > Доверие сертификатам > Включить доверие для Shadowrocket.

#### Пропуск прокси (skip-proxy)

Принудительно направляет трафик для указанных доменов или IP через TUN-интерфейс, а не через прокси. Используется для решения проблем совместимости.

```
skip-proxy = 192.168.0.0/16,10.0.0.0/8,172.16.0.0/12,localhost,*.local,captive.apple.com
```

#### Исключения маршрутов TUN (tun-excluded-routes)

TUN-интерфейс обрабатывает только TCP. Эта опция позволяет обойти указанные IP-диапазоны для других протоколов.

```
tun-excluded-routes = 10.0.0.0/8, 100.64.0.0/10, 127.0.0.0/8, 169.254.0.0/16, 172.16.0.0/12, 192.0.0.0/24, 192.0.2.0/24, 192.88.99.0/24, 192.168.0.0/16, 198.51.100.0/24, 203.0.113.0/24, 224.0.0.0/4, 255.255.255.255/32, 239.255.255.250/32, ff02::fb/128
```

#### DNS-сервер (dns-server)

Замена системного DNS. Поддерживаются обычный и зашифрованный DNS (DoH, DoH3, DoQ, DoT). Укажите `system` для использования системного DNS.

Обычный DNS:
```
dns-server = 223.5.5.5,119.29.29.29
```

DNS-over-HTTPS (DoH):
```
dns-server = https://dns.alidns.com/dns-query
```

DNS-over-HTTP/3 (DoH3):
```
dns-server = h3://dns.alidns.com/dns-query
```

DNS-over-QUIC (DoQ):
```
dns-server = quic://223.5.5.5
```

DNS-over-TLS (DoT):
```
dns-server = tls://223.5.5.5
```

Пересылка DNS через прокси (dns over proxy):
```
dns-server = https://dns.google/dns-query#proxy=server1
dns-server = https://dns.google/dns-query#ecs=120.76.0.0/14|2620:149:af0::10/56&ecs-override=true
dns-server = https://dns.google/dns-query#proxy=name&ecs=1.1.0.0/14|2620:149:af0::10/56&ecs-override=true
```

Параметры проксирования DNS:
- `proxy=name` — прокси-сервер (имя в URL-кодировке)
- `ecs=подсеть` — EDNS Client Subnet для передачи информации о подсети
- `ecs-override=true` — принудительное использование указанной подсети ECS

#### Резервный DNS (fallback-dns-server)

Используется если основной DNS недоступен или запрос превышает 2 секунды. `system` = системный DNS.

```
fallback-dns-server = system
```

#### IPv6 (ipv6, prefer-ipv6)

- `ipv6 = true/false` — включение поддержки IPv6. При true одновременно запрашиваются A и AAAA, приоритет у IPv4
- `prefer-ipv6 = true/false` — приоритетный запрос AAAA и использование IPv6

#### Системный DNS для DIRECT (dns-direct-system)

При true использует системный DNS для запросов доменов с правилом DIRECT.

```
dns-direct-system = false
```

#### ICMP (icmp-auto-reply)

Автоматический ответ на ping-запросы.

```
icmp-auto-reply = true
```

#### Всегда REJECT для перезаписи URL (always-reject-url-rewrite)

При false стратегия REJECT при перезаписи URL действует только в режиме конфигурации. При true — во всех режимах глобальной маршрутизации.

```
always-reject-url-rewrite = false
```

#### Частные IP (private-ip-answer)

При false, если домен возвращает частный IP, Shadowrocket считает домен захваченным и направляет через прокси. При true — пропускает.

```
private-ip-answer = true
```

#### Fallback на прокси для DIRECT (dns-direct-fallback-proxy)

При true, если не удалось разрешить DIRECT-домен, используется прокси.

```
dns-direct-fallback-proxy = true
```

#### TUN включённые маршруты (tun-included-routes)

Добавляет меньшую таблицу маршрутизации для TUN, чтобы часть трафика не терялась из-за меньшего размера маршрута Wi-Fi.

```
# tun-included-routes =
```

#### Всегда реальный IP (always-real-ip)

При true Shadowrocket возвращает реальный IP при обработке DNS через TUN, а не подставной. Используется для доменов, которые не поддерживают Fake-IP.

```
always-real-ip = time.*.com,ntp.*.com,*.cloudflareclient.com
```

#### Перехват DNS (hijack-dns)

Перехватывает DNS-запросы к жёстко заданным DNS-серверам (например Netflix использует 8.8.8.8).

```
hijack-dns = 8.8.8.8:53,8.8.4.4:53
hijack-dns = :53
```

Формат: `IP:порт`. Можно перехватить все DNS-запросы через `:53`.

#### Поведение UDP без поддержки (udp-policy-not-supported-behaviour)

Что делать с UDP-трафиком, попавшим на узел без поддержки UDP.
- `DIRECT` — прямая передача
- `REJECT` — отказ

```
udp-policy-not-supported-behaviour = REJECT
```

#### Включение конфигурации (include)

Включает содержимое другого файла. Приоритет текущей конфигурации выше.

```
include = base.conf
```

#### STUN-ответ (stun-response-ip, stun-response-ipv6)

Возврат подставного IP для предотвращения утечки реального IP через WebRTC.

```
# stun-response-ip = 1.1.1.1
# stun-response-ipv6 = ::1
```

#### Режим совместимости (compatibility-mode)

Значение 3 эквивалентно: Настройки > Прокси > Тип прокси > None.

```
# compatibility-mode =
```

#### Всегда локальный IP (always-ip-address)

При true принудительно использует локальный DNS для всех доменов. Скрытый параметр, может нарушить CDN.

```
# always-ip-address =
```

#### DNS для узлов (proxy-dns-server)

DNS для разрешения доменов узлов. По умолчанию через dns-server, для DNS-over-PROXY через системный DNS.

```
# proxy-dns-server =
```

#### Закрытие при потере цепи (close-if-proxy-chain-missing)

При true, если промежуточный узел в прокси-цепи потерян, соединение отклоняется. При false — пропускает потерянный узел.

```
# close-if-proxy-chain-missing =
```

#### IPv6 only (ipv6-only-if-no-ipv4-dns)

При true, если устройство получает только IPv6 DNS без IPv4, сеть считается IPv6 Only.

```
# ipv6-only-if-no-ipv4-dns =
```

#### Блокировка QUIC (block-quic)

- `all-proxy` — блокировать QUIC только для прокси
- `all` — блокировать все QUIC
- `always-allow` — разрешить всегда

```
block-quic = all-proxy
```

#### Локальный HOST для прокси (use-local-host-item-for-proxy)

При true использует локальное DNS-отображение для прокси вместо удалённого.

```
# use-local-host-item-for-proxy =
```

#### Все DNS-запросы (allow-dns-all)

При true: A/AAAA возвращают Fake-IP, остальные пересылаются вышестоящему DNS. При false: HTTPS, SVCB, TXT и т.д. возвращают пустой ответ. По умолчанию включено.

```
# allow-dns-all =
```

#### DNS SVCB (allow-dns-svcb)

При true разрешает SVCB-запросы. Система может выполнять SVCB вместо A-запросов, что мешает Fake-IP. По умолчанию запрещён.

```
# allow-dns-svcb =
```

#### Обход системы (bypass-system)

При true трафик самой системы (обновления и т.п.) обходит прокси.

```
bypass-system = true
```

#### URL обновления подписки (update-url)

URL для автоматического обновления подписки.

```
update-url = https://example.com/config.conf
```

---

### [Proxy] — узлы

Добавление локальных узлов (для совместимости с конфигурационными файлами, предпочтительнее подписки).

#### Shadowsocks

```
имя = ss,адрес,порт,password=пароль,method=aes-256-cfb,obfs=websocket,plugin=none
```

#### VMess

```
имя = vmess,адрес,порт,password=пароль,alterId=0,method=auto,obfs=websocket,tfo=1
```

#### VLESS

```
имя = vless,адрес,порт,password=пароль,tls=true,obfs=websocket,peer=example.com
```

#### HTTP

```
имя = http,адрес,порт,пользователь,пароль
```

#### HTTPS

```
имя = https,адрес,порт,пользователь,пароль
```

#### SOCKS5

```
имя = socks5,адрес,порт,пользователь,пароль
```

#### SOCKS5 over TLS

```
имя = socks5-tls,адрес,порт,пользователь,пароль,skip-common-name-verify=true
```

#### Trojan

```
имя = trojan,адрес,порт,password=пароль,allowInsecure=1,peer=example.com
```

#### Hysteria

```
имя = hysteria,адрес,порт,auth=пароль,obfsParam=запутывание,protocol=протокол,udp=1,peer=example.com,alpn=h2,upmbps=100,downmbps=100
```

#### Hysteria2

```
имя = hysteria2,адрес,порт,auth=пароль,obfsParam=запутывание,udp=1,peer=example.com,alpn=h3
```

#### TUIC

```
имя = tuic,адрес,порт,password=пароль,udp=1,user=uuid,peer=example.com,alpn=h2
```

#### Juicity

```
имя = juicity,адрес,порт,password=пароль,udp=1,user=uuid,peer=example.com,alpn=h2
```

#### WireGuard

```
имя = wireguard,адрес,порт,privateKey=приватный_ключ,publicKey=публичный_ключ,ip=IP_подсети,udp=1,dns=1.1.1.1,mtu=1350,keepalive=40,reserved=1/2/3
```

#### Snell

```
имя = snell,адрес,порт,password=пароль,udp=1,obfs=http,obfs-host=example.com,obfs-uri=/abc
```

---

### [Proxy Group] — группы прокси

#### Типы групп

- `select` — ручной выбор узла
- `url-test` — автоматическое переключение на узел с наименьшей задержкой
- `fallback` — автоматическое переключение на другой доступный узел при сбое текущего
- `load-balance` — распределение запросов разных правил по разным узлам группы
- `random` — случайный выбор узла

#### Параметры групп

- `interval` — интервал между тестами (секунды)
- `timeout` — тайм-аут теста (секунды)
- `tolerance` — допуск, переключение только если новый результат лучше старого на указанную величину
- `url` — URL для тестирования
- `policy-select-name` — имя предвыбранного узла
- `select=N` — стратегия по умолчанию (0 — первая, 1 — вторая и т.д.)

#### Пример группы без фильтра

```
имя = select,direct,proxy,подписка,группа_прокси,узел,interval=600,timeout=5,tolerance=100,url=https://core.telegram.org
```

#### Пример группы с regex-фильтром

```
имя = url-test,policy-regex-filter=регулярное_выражение,interval=600,timeout=5,tolerance=100,url=https://core.telegram.org
```

#### Пример группы с фильтрацией подписки

```
имя = select,имя_подписки,use=true,policy-regex-filter=регулярное_выражение,interval=600,timeout=5,tolerance=100,url=https://core.telegram.org
```

#### policy-regex-filter — фильтрация по регулярному выражению

Сохранить узлы с ключевыми словами A и B:
```
(?=.*(A))^(?=.*(B))^.*$
```

Сохранить узлы с A или B:
```
A|B
```

Исключить узлы с A или B:
```
^((?!(A|B)).)*$
```

Сохранить узлы с A, исключив содержащие B:
```
(?=.*(A))^((?!(B)).)*$
```

---

### [Rule] — правила маршрутизации

#### Типы правил

| Тип | Формат | Описание |
|-----|--------|----------|
| DOMAIN-SUFFIX | `DOMAIN-SUFFIX,example.com,СТРАТЕГИЯ` | Сопоставление суффикса домена (`a.example.com`, `a.b.example.com`) |
| DOMAIN-KEYWORD | `DOMAIN-KEYWORD,exa,СТРАТЕГИЯ` | Ключевое слово в домене |
| DOMAIN-WILDCARD | `DOMAIN-WILDCARD,a*c.example*.com,СТРАТЕГИЯ` | Подстановочные знаки `{*, ?}` |
| DOMAIN | `DOMAIN,www.example.com,СТРАТЕГИЯ` | Полный домен, только точное совпадение |
| USER-AGENT | `USER-AGENT,MicroMessenger*,СТРАТЕГИЯ` | User-Agent с поддержкой `*` |
| URL-REGEX | `URL-REGEX,^https?://.+/item.+,СТРАТЕГИЯ` | Регулярное выражение URL |
| IP-CIDR | `IP-CIDR,192.168.1.0/24,СТРАТЕГИЯ` | IPv4/IPv6 диапазон |
| IP-CIDR (no-resolve) | `IP-CIDR,172.16.0.0/12,СТРАТЕГИЯ,no-resolve` | IP-правило без DNS-запроса для доменов |
| IP-ASN | `IP-ASN,56040,СТРАТЕГИЯ` | Номер ASN |
| RULE-SET | `RULE-SET,URL,СТРАТЕГИЯ` | Набор правил с типами |
| DOMAIN-SET | `DOMAIN-SET,URL,СТРАТЕГИЯ` | Набор доменов без типа правила |
| SCRIPT | `SCRIPT,имя_скрипта,СТРАТЕГИЯ` | Имя скрипта (тип rule) |
| DST-PORT | `DST-PORT,443,СТРАТЕГИЯ` | Порт назначения |
| GEOIP | `GEOIP,CN,СТРАТЕГИЯ` | GeoIP-база |
| FINAL | `FINAL,СТРАТЕГИЯ` | Стратегия по умолчанию |
| AND | `AND,((DOMAIN,www.example.com),(DST-PORT,123)),СТРАТЕГИЯ` | Логическое И |
| NOT | `NOT,((DST-PORT,123)),СТРАТЕГИЯ` | Логическое НЕ |
| OR | `OR,((DST-PORT,123),(DST-PORT,456)),СТРАТЕГИЯ` | Логическое ИЛИ |
| PROTOCOL | `PROTOCOL,UDP` | Протокол (UDP, TCP) |

#### Стратегии правил

| Стратегия | Описание |
|-----------|----------|
| PROXY | Через прокси-сервер |
| DIRECT | Напрямую, без прокси |
| TAILSCALE | Через Tailscale-туннель |
| REJECT | HTTP 404, без содержимого |
| REJECT-DICT | HTTP 200 и пустой JSON-объект `{}` |
| REJECT-ARRAY | HTTP 200 и пустой JSON-массив `[]` |
| REJECT-200 | HTTP 200, без содержимого |
| REJECT-IMG | HTTP 200 и GIF 1px |
| REJECT-TINYGIF | HTTP 200 и GIF 1px |
| REJECT-DROP | Отбрасывание IP-пакета |
| REJECT-NO-DROP | ICMP «порт недоступен» |

Также стратегией может быть имя группы прокси, имя подписки, имя группы или имя узла.

#### Приоритет правил

1. Правила модуля выше правил конфигурации
2. Правила применяются сверху вниз
3. Доменные правила (DOMAIN, DOMAIN-SUFFIX, DOMAIN-KEYWORD, DOMAIN-WILDCARD) выше IP-правил (IP-CIDR, IP-ASN, GEOIP)
4. При совпадении DOMAIN, DOMAIN-SUFFIX, DOMAIN-WILDCARD и DOMAIN-KEYWORD срабатывает только один тип

#### Примеры

```
RULE-SET,https://raw.githubusercontent.com/tatarinovms/SRCFG/master/lists/ruads.list,REJECT
RULE-SET,https://raw.githubusercontent.com/tatarinovms/SRCFG/master/lists/rudirect.list,DIRECT
RULE-SET,https://raw.githubusercontent.com/tatarinovms/SRCFG/master/lists/ai.list,AUTO
GEOIP,RU,DIRECT
FINAL,DIRECT
```

#### Блокировка QUIC через правило

```
AND,((PROTOCOL,UDP),(DST-PORT,443)),REJECT-NO-DROP
```

---

### [Host] — привязки доменов

Привязка домена к IP:
```
example.com = 1.2.3.4
```

Назначение DNS-сервера для домена:
```
example.com = server:1.2.3.4
```

Назначение DNS-сервера для Wi-Fi (несколько DNS через запятую):
```
ssid:имя_wifi = server:1.2.3.4
```

Использование системного DNS для домена:
```
*.apple.com = server:system
localhost = 127.0.0.1
```

---

### [URL Rewrite] — перезапись URL

Формат: `регулярное_выражение URL_замены код_ответа`

```
^https?://(www.)?g.cn https://www.google.com 302
^https?://(www.)?google.cn https://www.google.com 302
```

Коды ответа: 301, 302 и др.

---

### [MITM] — расшифровка HTTPS

Shadowrocket расшифровывает только трафик доменов из `hostname`. Подстановочные знаки поддерживаются. Префикс `-` для исключения.

С версии 2.2.81 (3096) поддерживается MITM через HTTP/2 (`h2 = true`).

```
hostname = *.google.cn
hostname = -*.example.com,*.target.com
```

Внимание: расшифровка `*.apple.com`, `*.icloud.com` может вызвать проблемы из-за строгой политики безопасности iOS.
