## Какой системный вызов делает команда cd?

```
$ strace /bin/bash -c 'cd /tmp'
...
chdir("/tmp")
...
```
Команда `cd` делает системный вызов `chdir()`

---

## Используя strace выясните, где находится база данных file на основании которой она делает свои догадки.

Судя по выводу strace, источником данных служит `/etc/magic`.
`man file` говорит что это действительно так. Кроме того, согласно man, команде `file` можно эксплицитно указать скомпилированный magic файл, так же есть дефолтный порядок поиска этих файлов.

---

## Предложите способ обнуления открытого удаленного файла

```
$ lsof +L1 # вернет список удаленных файлов с PID процесса их запустившего и их FD(файловым дескриптором)
$ > /proc/$PID/fd/$FD # обнулит файл
```
Данный трюк сработает, только если файл был открыт в режиме append. Обнулить таким образом файл, открытый в режиме write у меня не получилось. Если есть способ обнулить и такой файл без остановки процесса, дайте, пожалуйста, знать в комментариях к ДЗ.

---

## Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Согасно википедии
> all of the memory and resources associated with it are deallocated so they can be used by other processes. However, the process's entry in the process table remains.

Значит, короткий ответ: нет, не занимают. Строго говоря, это логично, так как процесс, по-сути, завершен. Но проблема в том, что зомби-процесс занимает место в ограниченной по размеру в таблице процессов.

---

## На какие файлы вы увидели вызовы группы `open` за первую секунду работы `opensnoop` утилиты?

Если я правильно понял задание

```
$ strace -o snoop_output /usr/sbin/opensnoop-bpfcc # запишем вывод strace по работе утилиты в файл snoop_output
$ head -500 snoop_output | grep open
> openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libdl.so.2", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libutil.so.1", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libm.so.6", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libexpat.so.1", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
  openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
  openat(AT_FDCWD, "/usr/bin/pyvenv.cfg", O_RDONLY) = -1 ENOENT (No such file or directory)
  openat(AT_FDCWD, "/usr/pyvenv.cfg", O_RDONLY) = -1 ENOENT (No such file or directory)
  openat(AT_FDCWD, "/etc/localtime", O_RDONLY|O_CLOEXEC) = 3
  ....
```

---

## Какой системный вызов использует uname -a?
```
$ strace uname -a
...
uname({sysname="Linux", nodename="vagrant", ...}) = 0
...

$ man 2 uname
...
Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
...
```

---

## Чем отличается последовательность команд через ; и через && в bash?

- оператор `;` выполняет команды последовательно одну за другой. Поэтому в первом примере мы видим вывод строки `Hi`
- оператор `&&` выполняет команду последовательно, но только в том случае, если предыдущие команды завершаются успехом, т.е. exit status предыдущей команды - 0. Поэтому мы не видим вывода на экран, т.к. `test -d /tmp/some_dir` не вернет 0 в exit status, пока такой директории не существует.

Наверное, нет смысла в использовании никаких операторов при установк `set -e`, потому как при не нулевом статусе команды будет произведен выход из оболочки.

---

## Из каких опций состоит режим `bash set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?

- `e` - выход из оболочки при exit status любой команды >0
- `u` - ссылка на ранее не определенную переменную приведет к ошибке
- `x` - по каждой команде будут записаны ошибки
- `o pipefail` - параметр предотвращает маскировку ошибок в pipeline. Если какая-либо команда в pipeline завершается неудачно, этот exit status/code будет использоваться в качестве exit status/code всего pipeline.

Данный режим полезно использовать в скриптах, т.к. при возникновении проблем/ошибок сразу станет понятно что произошло. Если не использовать подобных конструкций, то скрипт может упасть, а мы об этом ничего не узнаем.

---

## Kакой наиболее часто встречающийся статус у процессов в системе?

В моей виртуальной машине в моменте вышло так что процессов типа `S` (interruptible sleep (waiting for an event to complete)) и процессов типа `I` (Idle kernel thread), со всеми их "подсемействами" было равное количество: по 43. 
Команду `ps` я запукал с ключами `axt`.

Согласно man дополнительные буквы означают:
- < - высокий приотритет
- N - низкий приоритет
- L - имеет страницы, заблокированные в памяти
- s - session leader
- l - многопоточный
- `+` - foreground process group
