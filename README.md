## Knock
программа для подлключения по ssh после выполнения входной последовательности портов

Port Knocking состоит из двух программ:
— сервер (knockd)
— клиент (knock)
Установим сервер:
```commandline
sudo apt-get install knockd
```
Далее просмотрим все сетевые порты:
```commandline
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    ...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    ...
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    ...
```
В данном случае eth0это интерфейс, который привязывается к общедоступному IP-адресу. Поэтому я буду использовать этот интерфейс для настройки knockd.

Далее отредактируйте knockdфайл конфигурации.
файл конфигурации вы можете найти по адресу: /usr/local/etc/knockd.conf
если его там нет используйте поиск:
```commandline
find / -name 'knockd.conf'
```
отредактируйте файл конфигурации по вашему усмотрению, установите свою последовательность портов, сохраните изменения:
```commandline
sudo nano /usr/local/etc/knockd.conf
sudo nano /etc/default/knockd
```
```commandline
# control if we start knockd at init or not
# 1 = start
# anything else = don't start
# PLEASE EDIT /etc/knockd.conf BEFORE ENABLING
START_KNOCKD=1

# command line options
KNOCKD_OPTS="-i eth1"
```
Запустим сервирную часть knockd:
```commandline
sudo systemctl start knockd
sudo systemctl enable knockd
sudo systemctl status knockd
```
если хотим изменить порт ssh допустим на 6036, раскоментируйте и поменяйти в файле конфигурации порт 22 на другой
```commandline
sudo nano /etc/ssh/sshd_config

после чего перезапустим службу

systemctl restart sshd.service
```
Сначала выполните следующую команду, чтобы она не разорвала наше текущее активное соединение:
```commandline
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Теперь выполните iptables-команду, чтобы заблокировать все входящие SSH-соединения на порту 60636по умолчанию:
```commandline
sudo iptables -A INPUT -p tcp --dport 60636 -j REJECT
или для ее отмены
sudo iptables -D INPUT -p tcp --dport 60636 -j REJECT
```



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
Теперь установим git и сборщики build-essential и gcc:
```commandline
sudo apt-get install git
sudo apt-get install build-essential
sudo apt-get install gcc make
```

#### Установка
Затем выполните следующее:
```commandline
git clone https://github.com/BEPb/knock
cd knock
autoreconf -fi
./configure --prefix=/usr/local
make
sudo make install
```
убедится в том что установка прошла успешно, можно набрав команду:
```commandline
knock -V
```
вы должны увидеть следующее:
```commandline
knock 1.0
Copyright (C) 2023 BEPb
```
так же для того что бы увидеть что программа запущена и работает:
```commandline
ps -ax | grep knock
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

