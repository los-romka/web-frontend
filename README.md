# HOWTO dive into web-frontend

## Термины в тексте

*host, Хост* - машина, на которой у нас стоит виртуалка

*vm, ВМ* - виртуалка (VirtualBox с Linux Ubuntu Server)

*daemon* - демон, сервис-служба, процесс, который работает в фоне и что-то делает

## Diving

### Настройка VM
Настроить ВМ так, чтобы из ВМ пинговался `google.com`:

```shell script
vm$ ping google.com
```

Если пакеты не теряются - значит всё ок!

Также нам нужно будет с хоста стучаться в ВМ, поэтому:

```shell script
vm$ ifconfig | grep inet
```
или 
```shell script
vm$ ip a | grep inet
```
Находим IP-шник `XXX.XXX.XXX.XXX` (127.0.0.1 - это не он, не тупи! U know it).
```shell script
host$ ping XXX.XXX.XXX.XXX
```

Если пакеты не теряются - значит всё ок!
Если что-то идёт не так - попробуйте в настройках виртуальной машины переключить Сетевой Адаптер в режим Моста (Bridge)

### WebStorm

Нам в дальнейшем нужно будет пользоваться нормальной IDE, поэтому скачиваем на хост [WebStorm](https://www.jetbrains.com/webstorm/), идём на кухню - ставим чайник, завариваем чай.

### Web-Server

Для полного погружения в разработку нам понадобится Веб-сервер. Мы будем использовать `apache2`. Устанавливаем через пакетный менеджер `apt`:
```shell script
vm$ apt install apache2
```
Заругается, мол вам не дозволено. Запустим ту же команду от имени "суперпользователя" (далее так будем делать всегда. ругается на что-то? - пробуем через `sudo`).

```shell script
vm$ sudo apt install apache2
```

Проверяем что `apache2` установился и запустился как демон (помним хитрость про `sudo`).
```shell script
vm$ service apache2 status
```

Должно быть `active (running)`

service - полезная вещь, позволяет посмотреть запущенных демонов.
```shell script
vm$ service --status-all
```

Если на весь экран вывод не поместился, то можно к выводу в стандартный поток `stdout` применить `|` (`pipe`) и какую нибудь утилиту с поддержкой scroll'а. Например `less` или `more`
```shell script
vm$ service --status-all | less
vm$ service --status-all | more
```

Проверим, что всё ок введя в браузере хоста адрес (`XXX.XXX.XXX.XXX` - IP-шник виртуалки):
```
http://XXX.XXX.XXX.XXX/
```

Приветственная страничка `apache2`? - УСПЕХ!

### Делаем свою первую web-страницу

Попробуем заменить приветствие на своё самым простым методом - перезапишем:
```shell script
vm$ echo "Hello world" > /var/www/html/index.html
```
Заругается тк недостаточно прав - станем владельцем файла (не забываем про `sudo`, если вдруг)

Для начала узнаем кто мы такой(ая):
```shell script
vm$ whoami
```

Это наше имя. Поменяем владельца файла `index.html` на себя (всё ещё не забываем про `sudo`)

```shell script
vm$ chown НАШЕ_ИМЯ /var/www/html/index.html
```

или так. на место `$(whoami)` подставится имя текущего пользователя

```shell script
vm$ chown $(whoami) /var/www/html/index.html
```

снова пробуем

```shell script
vm$ echo "Hello world" > /var/www/html/index.html
```

Возвращаемся в браузер - радуемся!

Кстати чай уже остыл... И `WebStorm` скачался. Бежим на кухню за чаем, устанавливаем `WebStorm` (не забываем про тёмную тему - мы ж нинзя!).

### SSH-Server

Настоящие web-сервера не всегда являются виртуалками на хост-машине. Обычно они расопложены где-то далеко, вне нашего компьютера. Для управления таким web-сервером нам необходимым инструмент для удалённого подключения. Реализуем такое взаимодействие через защищённый протокол `SSH`.
 
Для начала установим ssh-сервер на виртуалке.

```shell script
vm$ apt install openssh-server
```

его демон называется `sshd`, вспоминаем как проверить состояние демона - проверяем.

### WebStorm + Uploading via SFTP

Настроим IDE так, чтобы файлы мы редактировали на хосте, а затем выгружали на ВМ.
 
Открываем `WebStorm`, создаём новый проект. Далее добавим "удалённый хост" (нашу вируталку): ```Tools -> Deployment -> Configuration -> + -> SFTP```

Обзовём это дело `vm`.

Заполняем настройки соединения:
* `Host: XXX.XXX.XXX.XXX` - ip-адрес нашей виртуалки
* `Port: YY` - порт SSH-Server'а, По умолчанию SSH-Server работает на 22 порту.
* `User name: ZZZZ` - имя пользователя на виртуалке, под чьими правами мы там будем "управлять"
* `Password: PPPP` - пароль пользователя
* `Root path: /var/www/html` - корневая директория web-сервера

Нажимаем `Test Connection` - Проверяем что всё ок.

Переходим во вкладку `Mappings` указываем `Deployment Path: /` - жмакаем OK

Нам необходимы права, чтобы добавлять новые файлы в директорию `/var/www/html` на виртуалке.
Для этого не забудьте поменять владельца этой директории.

Осталось совсем чуть-чуть. Слева в `WebStorm` нажимаем на директорию нашего проекта правой кнопкой мыши, далее `Deployment -> Sync` (может словесно отличаться от версии к версии)
IDE вычислит расхождения в файлах нашего проекта между хостом и директорией `/var/www/html` на виртуалке.

Чтобы синхронизировать состояние файлов - нажимаем `>>`. В нашем проекте должен появится файл `index.html`.
Отредактируйте его и повторите действия синхронизации.

Возвращаемся в браузер - радуемся!

*Кстати, не забудьте про чай!*
