## Домашнее задание к занятию «2.4. Инструменты Git»
### Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.
```
> git log -1 --pretty=oneline aefea
> git log -1 --pretty='%H %s' aefea
> git show -s --pretty=oneline aefea
> git show -s --pretty='%H %s' aefea


aefead2207ef7e2aa5dc81a34aedf0cad4c32545 Update CHANGELOG.md
```
---
### Какому тегу соответствует коммит 85024d3?
```
> git log -1 --pretty=%s 85024d3
> git show -s --pretty=%s 85024d3

v0.12.23
```
---
### Сколько родителей у коммита b8d720? Напишите их хеши.
> Ответ: 2
```
> git show -s --pretty=%P b8d720
> git log -1 --pretty=%P b8d720

56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b
```
---
### Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.
```
> git show -s --oneline v0.12.23..v0.12.24
> git show -s --pretty='%h %s' v0.12.23..v0.12.24
> git log --oneline v0.12.23...v0.12.24
> git log --pretty='%h %s' v0.12.23...v0.12.24

33ff1c03b v0.12.24
b14b74c49 [Website] vmc provider links
3f235065b Update CHANGELOG.md
6ae64e247 registry: Fix panic when server is unreachable
5c619ca1b website: Remove links to the getting started guide's old location
06275647e Update CHANGELOG.md
d5f9411f5 command: Fix bug when using terraform login on Windows
4b6d06cc5 Update CHANGELOG.md
dd01a3507 Update CHANGELOG.md
225466bc3 Cleanup after v0.12.23 release
```
---
### Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).
```
> git log -S 'func providerSource' --pretty='%h'
> git log -G '^\s*func providerSource[(].*).*$' --pretty='%h'

5af1e6234 (тут функция была переписана)
8c928e835 (тут функция была впервые определена)

> git show 8c928e835
```
---
### Найдите все коммиты в которых была изменена функция globalPluginDirs.
Строго говоря, таких нет, есть только один коммит, где функция была определена, остальное - ее вызовы

Если конкретнее, то тут она была определена
```
> git log -G '^\s*func globalPluginDirs[(].*).*$' --pretty='%h'

8364383c3
```
А тут ее вызововы вместе с определением
```
git log -G '^.*globalPluginDirs[(].*).*$' --pretty='%h'

22a2580e9                                                                                                                                                      │
35a058fb3                                                                                                                                                      │
8364383c3
```
И даже если искать все совпадения, то найдется еще один коммит, в котором название функции фигурирует внутри комментария
```
>git log -G globalPluginDirs --pretty='%h'

22a2580e9                                                                                                                                                      │
35a058fb3                                                                                                                                                      │
c0b176109                                                                                                                                                      │
8364383c3
```

И да, я посмотрел diff всех этих коммитов))

И да, git blame говорит что она не менялась.

Все это справедливо, если под изменением функции подразумевается ее переопределение.

---
### Кто автор функции synchronizedWriters?
```
> git log -G "^\s*func synchronizedWriters(.*).*$" --pretty=%an


Martin Atkins
```
