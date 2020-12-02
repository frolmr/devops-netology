## Мы выгрузили JSON, который получили через API запрос к нашему сервису. Нужно найти и исправить все ошибки, которые допускает наш сервис.

Исправленный JSON (с точки зрения синтаксиса)

```json
{
   "info":"Sample JSON output from our service\t",
   "elements":[
      {
         "name":"first",
         "type":"server",
         "ip":7175
      },
      {
         "name":"second",
         "type":"proxy",
         "ip":"71.78.22.43"
      }
   ]
}
```
---

## К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: { "имя сервиса" : "его IP"}. Формат записи YAML по одному сервису: - имя сервиса: его IP. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

```python
#!/usr/bin/env python3

import os
import socket
import ast
import json
import yaml

dns_records = {'drive.google.com', 'mail.google.com', 'google.com'}

res = {}

for rec in dns_records:
    addr = socket.gethostbyname(rec)
    print(f'{rec} - {addr}')
    res[rec] = addr

ips_database = './database'

is_changed = False

if os.path.exists(ips_database):
    with open(ips_database) as f:
        data = f.read()
        prev = ast.literal_eval(data)

    for rec in prev:
        if prev[rec] != res[rec]:
            is_changed = True
            print(f'[ERROR] {rec} IP mismatch: {prev[rec]} {res[rec]}')

with open(ips_database, 'w') as file:
    print(res, file=file)

json_file = './ips.json'
yaml_file = './ips.yaml'

if is_changed or not os.path.exists(json_file) or not os.path.exists(yaml_file):
    hosts_list = []
    for rec in res:
        hosts_list.append({ rec: res[rec] })

    with open(yaml_file, 'w') as file:
        yaml.dump(hosts_list, file)

    with open(json_file, 'w') as file:
        json.dump(hosts_list, file)

```
таким образом в результатом работы скрипта будут в том числе 2 файла: `ips.json` и `ips.yaml` со следующим содержимым соответсвенно:
```json
[{"mail.google.com": "74.125.205.17"}, {"google.com": "64.233.165.101"}, {"drive.google.com": "64.233.165.194"}]
```

```yaml
- mail.google.com: 74.125.205.17
- google.com: 64.233.165.101
- drive.google.com: 64.233.165.194
```

---
