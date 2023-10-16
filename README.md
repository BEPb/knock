## Knock
программа для подлключения по ssh после выполнения входной последовательности портов


### Сборка

#### Проверка
Чтобы собрать Knockd, убедитесь, что у вас есть libpcap и инструменты autoconf установлен. 
Для libpcap:
```commandline
dpkg -l | grep libpcap
```
Если libpcap установлен, вы увидите строку, которая начинается с ii и которая указывает имя пакета, версию и 
описание. Если его нет, то ничего не будет выведено. 
Для autoconf:
```commandline
dpkg -l | grep autoconf
```
Аналогично предыдущему, если autoconf установлен, вы увидите строку, начинающуюся с ii, соответствующую этому пакету.
Если его нет, то ничего не будет выведено. 

#### Подготовка
Если пакеты не были установлены, то для установки libpcap:
Введите следующую команду в терминал:
```commandline
sudo apt-get install libpcap-dev
```
Установка autoconf:
Введите следующую команду в терминал:
```commandline
sudo apt-get install autoconf
```
После ввода команды sudo apt-get install ..., вам может потребоваться ввести пароль администратора для подтверждения 
установки. После завершения установки вы сможете использовать libpcap и autoconf в вашей системе 

#### Установка
Затем выполните следующее:
```commandline
autoreconf -fi
./configure --prefix=/usr/local
make
sudo make install
```

### ПРИМЕР
Приведенный ниже пример можно использовать для запуска брандмауэра со строгой политикой подключения, доступ к которому 
возможен только после успешного выполнения последовательного подключения по разным портам.

   1. Клиент отправляет четыре пакета TCP SYN на сервер по следующим портам:
      38281, 29374, 4921, 54918
   2. Сервер обнаруживает это и запускает команду iptables, чтобы открыть порт 22 для клиента.
   3. Клиент подключается к серверу через SSH и делает все, что ему нужно.
   4. Клиент отправляет на сервер еще четыре TCP SYN-пакета: 
     37281, 8529, 40127, 10100
   5. Сервер обнаруживает это и запускает другую команду iptables, чтобы закрыть порт 22 к клиенту.

