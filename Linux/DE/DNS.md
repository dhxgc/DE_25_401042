## DNS

---

Временно выставляем другой DNS-server на HQ-SRV, удялаем предыдущие строки.
`/etc/resolv.conf`:
```
nameserver 8.8.8.8
```
После скачиваем DNS-сервер BIND:
```
apt install bind9 -y
```
Редактируем файл `/etc/bind/named.conf.options`
Файл должен выглядеть следующим образом:
![[Pasted image 20241031221924.png]]
Все, что выделено белым - добавили мы. `//` - просто комментарий.

>- **listen-on { any; };**: Этот параметр определяет адреса и порты, на которых DNS-сервер будет слушать запросы. Значение any означает, что сервер будет прослушивать запросы на всех доступных интерфейсах и IP-адресах.
>- **recursion yes;**: Устанавливает, разрешено ли серверу делать рекурсивные запросы. Рекурсивные запросы возникают, когда DNS-сервер запрашивает другие серверы и выполняет несколько итераций поиска до тех пор, пока не получит окончательный ответ.
>- **allow-query { any; };**: Определяет список IP-адресов или подсетей, которым разрешено отправлять запросы на этот DNS-сервер. Значение any позволяет принимать запросы от всех клиентов.
>- **forwarders { 77.88.8.8; };**: Этот параметр используется для настройки сервера DNS в режим "пересылки" (forwarding). Когда сервер получает запрос на имя, которое он не может разрешить (не имеет информации в своих зонах), он может перенаправить запрос на другой DNS-сервер (forwarder), указанный в этой директиве.

Далее перезагружаем:
```bash
systemctl --now enable named; systemctl status named; 
```

Приводим `/etc/resolv.conf` к следующему виду:
```
search au-team.irpo
nameserver 127.0.0.1
```

Проверяем разрешение имен на данном этапе:
```bash
root@kk:/home/kk# ping ya.ru
PING ya.ru (77.88.44.242) 56(84) bytes of data.
64 bytes from 77.88.44.242: icmp_seq=1 ttl=55 time=24.6 ms
64 bytes from 77.88.44.242: icmp_seq=2 ttl=55 time=24.7 ms
64 bytes from 77.88.44.242: icmp_seq=3 ttl=55 time=24.7 ms

--- ya.ru ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2052ms
rtt min/avg/max/mdev = 24.587/24.670/24.736/0.062 ms
```

В `/etc/bind/named.conf.local` добавляем:
```
zone "au-team.irpo" {
        type master;
        file "/etc/bind/au-team.db";
}
```

Создаем файл с настройками прямой зоны и выставляем нужные права:
```bash
touch /etc/bind/au-team.db
chown bind:bind /etc/bind/au-team.db
chmod 600 /etc/bind/au-team.db
```

Шаблоны доступны на сайте [официальной документации.](https://wiki.debian.org/Bind9) Мы берем следующее содержимое и вставляем в файл `au-team.db`:

![[Pasted image 20241101000406.png]]

После прописываем обратные зоны - их 3, также в файле `/etc/bind/named.conf.local`

```
zone "100.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/au-team100rev.db";
}

zone "200.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/au-team200rev.db";
}
```

Создаем файлы, выдаем права:
```bash
touch /etc/bind/au-team{100,200}rev.db
chown bind:bind /etc/bind/au-team{100,200}rev.db
chmod 600 /etc/bind/au-team{100,200}rev.db
```

#### `/etc/bind/au-team100rev.db`:

```
$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                                2024102200      ; serial
                                12H             ; refresh
                                1H              ; retry
                                1W              ; expire
                                1H              ; ncache
                        )
        IN      NS      au-team.irpo.
1       IN      PTR     hq-rtr.au-team.irpo.
10      IN      PTR     hq-srv.au-team.irpo.
```

#### `/etc/bind/au-team200rev.db`:

```
$TTL    1D
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                                2024102200      ; serial
                                12H             ; refresh
                                1H              ; retry
                                1W              ; expire
                                1H              ; ncache
                        )
        IN      NS      au-team.irpo.
2       IN      PTR     hq-cli.au-team.irpo.
```

Проверяем, перезагружаем `bind`: 

```bash
named-checkconf -z
systemctl restart named
```

#### Скринов с проверкой нет(
