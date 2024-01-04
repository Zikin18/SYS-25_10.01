# Домашнее задание к занятию «Disaster recovery и Keepalived» Белобородов Юрий

### Цель задания
В результате выполнения этого задания вы научитесь:
1. Настраивать отслеживание интерфейса для протокола HSRP;
2. Настраивать сервис Keepalived для использования плавающего IP


### Чеклист готовности к домашнему заданию

1. Установлена программа Cisco Packet Tracer
2. Установлена операционная система Ubuntu на виртуальную машину и имеется доступ к терминалу
3. Сделан клон этой виртуальной машины, они находятся в одной подсети и имеют разные IP адреса
4. Просмотрены конфигурационные файлы, рассматриваемые на лекции, которые находятся по [ссылке](https://github.com/netology-code/sflt-homeworks/blob/main/1/)


### Задание 1
- Дана [схема](https://github.com/netology-code/sflt-homeworks/blob/main/1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

Ответ:

![router1showTrack.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/1-1_router1showTrack.png)
![router2showTrack.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/1-2_router2showTrack.png)
![S0-R1-disconnect-state.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/1-3_S0-R1-disconnect-state.png)
![disconnected.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/1-4_disconnected.png)

[hsrp_advanced_updated.pkt](https://github.com/Zikin18/SYS-25_10.01/blob/master/hsrp_advanced_updated.pkt)


### Задание 2
- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](https://github.com/netology-code/sflt-homeworks/blob/main/1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

Ответ:

Скрипт для проверки состояния

`checkAlive.sh`

```
#!/bin/bash
exec 3> /dev/tcp/localhost/80
if [ $? -eq 0 ] && [ -f "/var/www/html/index.nginx-debian.html" ] ; 
then exit 0;
else exit 1;
fi
```

Конфигурационные файлы для Keepalived

Конфиг для MASTER

```
vrrp_script checkAlive{
    script "/home/vm1/checkAlive.sh"
    interval 3
}

vrrp_instance VI_1 {
        state MASTER
        interface enp0s8
        virtual_router_id 15
        priority 255
        advert_int 1

        virtual_ipaddress {
              192.168.56.115/24
        }

		track_script {
           checkAlive
        }

}
```

Конфиг для BACKUP

```
vrrp_script checkAlive{
    script "/home/vm1/checkAlive.sh"
    interval 3
}

vrrp_instance VI_1 {
        state BACKUP
        interface enp0s3
        virtual_router_id 15
        priority 200
        advert_int 1

        virtual_ipaddress {
              192.168.56.115/24
        }

	track_script {
           checkAlive
        }
}

```

![2-1_normal.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/2-1_normal.png)
![2-2_movefile.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/2-2_movefile.png)
![2-3_movefile_result.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/2-3_movefile_result.png)
![2-4_moveback_sysstate.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/2-4_moveback_sysstate.png)
![2-5_stop_nginx.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/2-5_stop_nginx.png)
![2-6_stop_nginx_result.png](https://github.com/Zikin18/SYS-25_10.01/blob/master/2-6_stop_nginx_result.png)
