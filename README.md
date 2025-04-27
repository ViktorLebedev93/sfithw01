# Домашнее задание к занятию 1 "`Disaster recovery и Keepalived`" - `Лебедев Виктор`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.

---

### Задание 1

1. `Дана схема для Cisco Packet Tracer, рассматриваемая в лекции.`
2. `На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)`
3. `Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).`
4. `Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.`
4. `На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.`

### Решение 1

**Скриншоты с настройками **
<img src="img/img1.jpg">
<img src="img/img2.jpg">
<img src="img/img3.jpg">
<img src="img/img4.jpg">
<img src="img/img5.jpg">

<a href="hsrp_advanced.pkt">Приложенный PKT файл</a>

---

### Задание 2

1. `Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного файла.`
2. `Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах`
3. `Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.`
4. `Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script`
5. `На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html`

### Решение 2
**1. Устанавливаем keepalived и запускаем http сервер на обоих нодах **
```
sudo apt install keepalived
python3 -m http.server 80 --bind 0.0.0.0

```
**2. Создаем скрипт проверки доступности сервера**
```
#!/bin/bash

WEB_PORT=80
WEB_ROOT="/home/viktor/http"
INDEX_FILE="index.html"

nc -z localhost $WEB_PORT
if [ $? -ne 0 ]; then
  echo "Port $WEB_PORT is down!"
  exit 1
fi

if [ ! -f "$WEB_ROOT/$INDEX_FILE" ]; then
  echo "$WEB_ROOT/$INDEX_FILE not exist!"
  exit 1
fi

exit 0
```
**3. Создаем keepalived.conf (нода1)**
```
vrrp_script chk_webserver {
    script "/home/viktor/checkserver.sh"
    interval 3
    weight 2
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens160 
    virtual_router_id 209
    priority 255
    advert_int 1
    virtual_ipaddress {
        10.1.170.209
    }
    track_script {
        chk_webserver
    }
}
```
**4. Создаем keepalived.conf (нода2)**
```
vrrp_script chk_webserver {
    script "/home/viktor/checkserver.sh"
    interval 3
    weight 2
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens160
    virtual_router_id 209
    priority 100
    advert_int 1
    virtual_ipaddress {
        10.1.170.209
    }
    track_script {
        chk_webserver
    }
}
```
**5. Скриншоты с демонстрацией **
<img src="img/img6.jpg">
<img src="img/img7.jpg">

