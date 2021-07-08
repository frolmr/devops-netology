## 1. Node exporter

```
$ sudo useradd --no-create-home --shell /bin/false node_exporter

$ wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz

$ tar -xvzf node_exporter-1.0.1.linux-amd64.tar.gz

$ cp node_exporter-1.0.1.linux-amd64/node_exporter /usr/local/bin/

$ sudo vim /etc/systemd/system/node_exporter.service

### inside vim
[Unit]
Description=Node Exporter

[Service]
User=node_exporter
Group=node_exporter
EnvironmentFile=-/etc/sysconfig/node_exporter
ExecStart=/usr/local/bin/node_exporter $OPTIONS

[Install]
WantedBy=multi-user.target
### EOF

$ sudo systemctl daemon-reload && \
  sudo systemctl start node_exporter && \
  sudo systemctl status node_exporter && \
  sudo systemctl enable node_exporter
```

Проверка показала, что все работает как надо: сервис живет, поднимается как надо (даже при ребуте виртуалки)

---

## 2. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

Для базового мониторинга я бы выбрал следующие опции:
- meminfo
- cpu
- filesystem
- netdev

С учетом unit файла из предыдущего пункта, можно сделать так
```
$ OPTIONS="--collector.meminfo --collector.cpu --collector.filesystem --collector.netdev"
```

---

## 4. Можно ли по выводу dmesg понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

```
$ dmesg -T | grep DMI
> [Sun Nov  1 16:31:30 2020] DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
```
Не уверен, что ОС в состоянии "осознать" это))) Но вывод говорит об этом. В случае запуска на не вирутальном хосте там будет что-нибудь вроде
```
> DMI: System x2530 M4: -[1214AC1]-/00Y8494, BIOS -[VVE134MUS-1.50]- 08/30/2013
```
---

## 5. Как настроен sysctl fs.nr_open на системе по-умолчанию?
```
$ sysctl -a | grep fs.nr_open
> fs.nr_open = 1048576 # This denotes the maximum number of file-handles a process can allocate

$ ulimit -n # the maximum number of open file descriptors
> 1024
```

---

## 6. Неймспейсы процессов

```
$ unshare -f --pid --mount-proc sleep 1h
$ ps aux | grep sleep
...
root        1760  0.0  0.0   8076   592 pts/3    S+   17:22   0:00 sleep 1h
...
$ nsenter --target 1760 --pid --mount
$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   8076   592 pts/3    S+   17:22   0:00 sleep 1h
root           2  0.0  0.4   9836  4108 pts/1    S    17:25   0:00 -bash
root          11  0.0  0.3  11484  3384 pts/1    R+   17:25   0:00 ps aux
```
---
## 7. Найдите информацию о том, что такое `:(){ :|:& };:`.

Это fork bomb. Происходит следующее определяется функция с именем `:`, которая дважды вызывает саму себя `: | :` в фоновом режиме `&`. Из-за рекурсивной природы ветвление будет продолжаться.
При отсутствии лимита на число процессов машина быстро исчерпывает физическую память и уходит в своп.

По умолчанию processes-per-user равен:
```
$ ulimit -u
> 3582
```
Можно подстраховать себя задав лимит на processes-per-user:
```
$ ulimit -u 50
```
будет действовать только для текущей сессии. Для постоянного ограничения можно отредактировать /etc/security/limits.conf: `* hard nproc 50`
