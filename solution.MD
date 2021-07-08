## Какие значения переменным c,d,e будут присвоены? Почему?

Имеется "скрипт":
```
a=1
b=2
c=a+b
d=$a+$b
e=$(($a+$b))
```

В переменную `c` будет записано `a+b`, т.к. оболочка воспринимает данное выражение как простую строку.

В переменную `d` будет щаписано `1+2`, т.к. оболочка так же воспринимает это как простую строку, только перед этим, она 'снимает' значения переменных `a` и `b`.

В переменную `e` будет записано `3`, т.к. оболочка мало того что разименовывает значения переменных `a` и `b`, так еще и интерпретирует перед записью в переменную результат их сложениия.

---

## Что необходимо сделать, чтобы исправить скрипт?

Есть скрипт:
```
while ((1==1)
do
curl https://localhost:4757
if (($? != 0))
then
date >> curl.log
fi
done
```

Строго говоря, скрипт вообще не должен запуститься, т.к. в первой строке отсутствует закрывающая скобка. В остальном скрипт выглядит рабочим.

---

## Необходимо написать скрипт, который проверяет доступность трёх IP: 192.168.0.1, 173.194.222.113, 87.250.250.242 по 80 порту и записывает результат в файл log. Проверять доступность необходимо пять раз для каждого узла.
```
hosts=(192.168.0.1 173.194.222.113 87.250.250.242)

for i in ${hosts[@]}
  do
    counter=0
    while (($counter < 5))
      do
        nc -w 1 -zv $i 80 &>>log

        let "counter += 1"
      done
    done
```

в результате запуска скрипта в файле `log` имеем следующий вывод
```
$ cat log

nc: connect to 192.168.0.1 port 80 (tcp) timed out: Operation now in progress
nc: connect to 192.168.0.1 port 80 (tcp) timed out: Operation now in progress
nc: connect to 192.168.0.1 port 80 (tcp) timed out: Operation now in progress
nc: connect to 192.168.0.1 port 80 (tcp) timed out: Operation now in progress
nc: connect to 192.168.0.1 port 80 (tcp) timed out: Operation now in progress
Connection to 173.194.222.113 80 port [tcp/http] succeeded!
Connection to 173.194.222.113 80 port [tcp/http] succeeded!
Connection to 173.194.222.113 80 port [tcp/http] succeeded!
Connection to 173.194.222.113 80 port [tcp/http] succeeded!
Connection to 173.194.222.113 80 port [tcp/http] succeeded!
Connection to 87.250.250.242 80 port [tcp/http] succeeded!
Connection to 87.250.250.242 80 port [tcp/http] succeeded!
Connection to 87.250.250.242 80 port [tcp/http] succeeded!
Connection to 87.250.250.242 80 port [tcp/http] succeeded!
Connection to 87.250.250.242 80 port [tcp/http] succeeded!
```

---

## Необходимо дописать скрипт из предыдущего задания так, чтобы он выполнялся до тех пор, пока один из узлов не окажется недоступным. Если любой из узлов недоступен - IP этого узла пишется в файл error, скрипт прерывается

```
hosts=(173.194.222.113 192.168.0.1 87.250.250.242)

for i in ${hosts[@]}
  do
    counter=0
    while (($counter < 5))
      do
        nc -w 1 -zv $i 80 &>>log

        if (($? != 0))
        then
          echo $i >> error
          break 2
        fi

        let "counter += 1"
      done
  done
```

В результате запуска в error запишется ip: `192.168.0.1`

---

## Git hooks

B `commit-message` добавляем такое:
```
MSG="$1"

if ! grep -qE "^\[.*\]" "$MSG";then
    cat "$MSG"
    echo "Your commit message must contain issue number in square brackets"
    exit 1
fi

while read line; do
    if [ "${line:0:1}" == "#" ]; then
        continue
    fi
    if [ ${#line} -ge 30 ]; then
        echo "Your commit message must be less than 30 characters"
        exit 1
    fi
done < "${1}"

exit 0

```
