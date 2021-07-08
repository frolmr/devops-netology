## Есть скрипт.
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

> Какое значение будет присвоено переменной c?

При такой реализации попытка присвоения c = a + b завершится ошибкой из-за несоответсвия типов в операции '+'

> Как получить для переменной c значение 12?

Надо явно привести значениие переменной `a` к строковому типу и провести конкатенацию строк
```python
c = str(a) + b # '12'
```
Если нас интересует именно целочисленное значение то привести получиившуюся строку `'12'` к числу, например так
```python
c = int(str(a) + b) # 12
```

> Как получить для переменной c значение 3?

Явно привести значение переменной `b` к числовому типу:

```python
c = a + int(b) # 3
```

---

## Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

Как один из возможных вариантов
```python
#!/usr/bin/env python3

import os

path = "~/netology/sysadm-homeworks"

bash_command = [f'cd {path}', "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(path + prepare_result)
```

в таком виде скрипт выводит все измененные файлы с полным путем

---

## Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

Один из вариантов рабочего скрипта

```python
#!/usr/bin/env python3

import os
import sys

path = sys.argv[1]

if not os.path.exists(path):
    print('No such directory')
    sys.exit()

if not os.path.isdir(f'{path}/.git'):
    print('Not a git repository!')
    sys.exit()

bash_command = [f'cd {path}', "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(path + prepare_result)

```

---

## Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: drive.google.com, mail.google.com, google.com.

```python
#!/usr/bin/env python3

import os
import socket
import ast

dns_records = {'drive.google.com', 'mail.google.com', 'google.com'}

res = {}
for rec in dns_records:
    addr = socket.gethostbyname(rec)
    print(f'{rec} - {addr}')
    res[rec] = addr

previous_ips_file = './prev_ips'

if os.path.exists(previous_ips_file):
    with open(previous_ips_file) as f:
        data = f.read()
        prev = ast.literal_eval(data)

    for rec in prev:
        if prev[rec] != res[rec]:
            print(f'[ERROR] {rec} IP mismatch: {prev[rec]} {res[rec]}')

with open(previous_ips_file, 'w') as file:
    print(res, file=file)

```

Таким образом при первом прогоне мы сгенерируем файл с адресами наших хостов, при последующих будем уведомлять если адрес изменился.
