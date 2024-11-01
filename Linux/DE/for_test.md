```
$TTL    86400
@       IN      SOA     au-team.irpo. admin.au-team.irpo. (
                        2023103101 ; Serial
                        3600       ; Refresh
                        1800       ; Retry
                        604800     ; Expire
                        86400      ; Negative Cache TTL
)

; NS Records
@       IN      NS      au-team.irpo.

; A Records
@       IN      A       192.168.100.10
hq-rtr   IN      A       192.168.100.1
br-rtr   IN      A       192.168.0.1
hq-srv   IN      A       192.168.100.10
br-srv   IN      A       192.168.0.10
hq-cli   IN      A       192.168.200.2

; CNAME Records
moodle   IN     CNAME    hq-rtr.
wiki     IN     CNAME    hq-rtr.

```