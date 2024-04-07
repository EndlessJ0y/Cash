# Домашнее задание к занятию "Система мониторинга Zabbix. Часть 2" - `Котлярова Дарья`


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
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1  

    Дана схема для Cisco Packet Tracer, рассматриваемая в лекции.
    На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
    Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
    Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
    На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

 


### Решение 1

	Схема https://github.com/EndlessJ0y/Disaster-recovery-Keepalived/blob/main/hsrp_advanced_HW.pkt (находится в репозитории)
	
![router 1](https://github.com/EndlessJ0y/Screens/blob/main/router1.jpeg)
![router 2](https://github.com/EndlessJ0y/Screens/blob/main/router2.jpeg)
### Задание 2

    Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного файла.
    Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
    Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
    Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
    На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html



## Решение 2 

![master ip](https://github.com/EndlessJ0y/Screens/blob/main/photo1712496106.jpeg)
![backup ip, unavailable nginx or absent index.html on master](https://github.com/EndlessJ0y/Screens/blob/main/photo1712496300.jpeg)

## Config of master


global_defs {
    enable_script_security
}

vrrp_script nginx_check {
    script "/usr/bin/curl -f http://127.0.0.1"
    interval 3
    user keepalived
}

vrrp_instance web {
    state MASTER
    interface enp1s0
    virtual_router_id 15
    priority 255
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass 12345
    }
    virtual_ipaddress {
        192.168.122.15/24
    }
    track_script {
        nginx_check
    }
}


## Config of backup


global_defs {
    enable_script_security
}

vrrp_script nginx_check {
    script "/usr/bin/curl -f http://127.0.0.1"
    interval 3
    user keepalived
}

vrrp_instance web {
    state BACKUP
    interface enp1s0
    virtual_router_id 15
    priority 100
    advert_int 2
    preempt_delay 30
    authentication {
        auth_type PASS
        auth_pass 12345
    }
    virtual_ipaddress {
        192.168.122.15/24
    }
    track_script {
        nginx_check
    }
}
