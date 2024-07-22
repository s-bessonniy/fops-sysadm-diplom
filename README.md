#  Курсовая работа на профессии "DevOps-инженер с нуля" - С. Яремко

Содержание
==========
* [Задача](#Задача)
* [Инфраструктура](#Инфраструктура)
    * [Сайт](#Сайт)
    * [Мониторинг](#Мониторинг)
    * [Логи](#Логи)
    * [Сеть](#Сеть)
    * [Резервное копирование](#Резервное-копирование)
    * [Дополнительно](#Дополнительно)
* [Выполнение работы](#Выполнение-работы)
* [Критерии сдачи](#Критерии-сдачи)
* [Как правильно задавать вопросы дипломному руководителю](#Как-правильно-задавать-вопросы-дипломному-руководителю) 

---------
## Задача
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в [Yandex Cloud](https://cloud.yandex.com/).

## Инфраструктура
Для развёртки инфраструктуры используйте Terraform и Ansible. 

Параметры виртуальной машины (ВМ) подбирайте по потребностям сервисов, которые будут на ней работать. 

Ознакомьтесь со всеми пунктами из этой секции, не беритесь сразу выполнять задание, не дочитав до конца. Пункты взаимосвязаны и могут влиять друг на друга.

### Сайт
Создайте две ВМ в разных зонах, установите на них сервер nginx, если его там нет. ОС и содержимое ВМ должно быть идентичным, это будут наши веб-сервера.

Используйте набор статичных файлов для сайта. Можно переиспользовать сайт из домашнего задания.

Создайте [Target Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/target-group), включите в неё две созданных ВМ.

Создайте [Backend Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/backend-group), настройте backends на target group, ранее созданную. Настройте healthcheck на корень (/) и порт 80, протокол HTTP.

Создайте [HTTP router](https://cloud.yandex.com/docs/application-load-balancer/concepts/http-router). Путь укажите — /, backend group — созданную ранее.

Создайте [Application load balancer](https://cloud.yandex.com/en/docs/application-load-balancer/) для распределения трафика на веб-сервера, созданные ранее. Укажите HTTP router, созданный ранее, задайте listener тип auto, порт 80.

Протестируйте сайт
`curl -v <публичный IP балансера>:80` 

### Мониторинг
Создайте ВМ, разверните на ней Zabbix. На каждую ВМ установите Zabbix Agent, настройте агенты на отправление метрик в Zabbix. 

Настройте дешборды с отображением метрик, минимальный набор — по принципу USE (Utilization, Saturation, Errors) для CPU, RAM, диски, сеть, http запросов к веб-серверам. Добавьте необходимые tresholds на соответствующие графики.

### Логи
Cоздайте ВМ, разверните на ней Elasticsearch. Установите filebeat в ВМ к веб-серверам, настройте на отправку access.log, error.log nginx в Elasticsearch.

Создайте ВМ, разверните на ней Kibana, сконфигурируйте соединение с Elasticsearch.

### Сеть
Разверните один VPC. Сервера web, Elasticsearch поместите в приватные подсети. Сервера Zabbix, Kibana, application load balancer определите в публичную подсеть.

Настройте [Security Groups](https://cloud.yandex.com/docs/vpc/concepts/security-groups) соответствующих сервисов на входящий трафик только к нужным портам.

Настройте ВМ с публичным адресом, в которой будет открыт только один порт — ssh. Настройте все security groups на разрешение входящего ssh из этой security group. Эта вм будет реализовывать концепцию bastion host. Потом можно будет подключаться по ssh ко всем хостам через этот хост.

### Резервное копирование
Создайте snapshot дисков всех ВМ. Ограничьте время жизни snaphot в неделю. Сами snaphot настройте на ежедневное копирование.

### Дополнительно
Не входит в минимальные требования. 

1. Для Zabbix можно реализовать разделение компонент - frontend, server, database. Frontend отдельной ВМ поместите в публичную подсеть, назначте публичный IP. Server поместите в приватную подсеть, настройте security group на разрешение трафика между frontend и server. Для Database используйте [Yandex Managed Service for PostgreSQL](https://cloud.yandex.com/en-ru/services/managed-postgresql). Разверните кластер из двух нод с автоматическим failover.
2. Вместо конкретных ВМ, которые входят в target group, можно создать [Instance Group](https://cloud.yandex.com/en/docs/compute/concepts/instance-groups/), для которой настройте следующие правила автоматического горизонтального масштабирования: минимальное количество ВМ на зону — 1, максимальный размер группы — 3.
3. В Elasticsearch добавьте мониторинг логов самого себя, Kibana, Zabbix, через filebeat. Можно использовать logstash тоже.
4. Воспользуйтесь Yandex Certificate Manager, выпустите сертификат для сайта, если есть доменное имя. Перенастройте работу балансера на HTTPS, при этом нацелен он будет на HTTP веб-серверов.

## Выполнение работы
На этом этапе вы непосредственно выполняете работу. При этом вы можете консультироваться с руководителем по поводу вопросов, требующих уточнения.

⚠️ В случае недоступности ресурсов Elastic для скачивания рекомендуется разворачивать сервисы с помощью docker контейнеров, основанных на официальных образах.

**Важно**: Ещё можно задавать вопросы по поводу того, как реализовать ту или иную функциональность. И руководитель определяет, правильно вы её реализовали или нет. Любые вопросы, которые не освещены в этом документе, стоит уточнять у руководителя. Если его требования и указания расходятся с указанными в этом документе, то приоритетны требования и указания руководителя.

## Критерии сдачи
1. Инфраструктура отвечает минимальным требованиям, описанным в [Задаче](#Задача).
2. Предоставлен доступ ко всем ресурсам, у которых предполагается веб-страница (сайт, Kibana, Zabbix).
3. Для ресурсов, к которым предоставить доступ проблематично, предоставлены скриншоты, команды, stdout, stderr, подтверждающие работу ресурса.
4. Работа оформлена в отдельном репозитории в GitHub или в [Google Docs](https://docs.google.com/), разрешён доступ по ссылке. 
5. Код размещён в репозитории в GitHub.
6. Работа оформлена так, чтобы были понятны ваши решения и компромиссы. 
7. Если использованы дополнительные репозитории, доступ к ним открыт. 

## Как правильно задавать вопросы дипломному руководителю
Что поможет решить большинство частых проблем:
1. Попробовать найти ответ сначала самостоятельно в интернете или в материалах курса и только после этого спрашивать у дипломного руководителя. Навык поиска ответов пригодится вам в профессиональной деятельности.
2. Если вопросов больше одного, присылайте их в виде нумерованного списка. Так дипломному руководителю будет проще отвечать на каждый из них.
3. При необходимости прикрепите к вопросу скриншоты и стрелочкой покажите, где не получается. Программу для этого можно скачать [здесь](https://app.prntscr.com/ru/).

Что может стать источником проблем:
1. Вопросы вида «Ничего не работает. Не запускается. Всё сломалось». Дипломный руководитель не сможет ответить на такой вопрос без дополнительных уточнений. Цените своё время и время других.
2. Откладывание выполнения дипломной работы на последний момент.


### Решение.
Для начала создаем файл main.tf для terraform для создания инфроструктуры в Yandex Cloud:

main.tf

```HCL
# -----Create infrostructure-----

#image_id = "fd85bll745cg76f707mq" webserver
#image_id = "fd8rp5pmb90k8l5o8php" bastion

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  token     = var.token
  cloud_id  = var.cloud_id
  folder_id = var.folder_id
}

#webservers
resource "yandex_compute_instance" "webserver" {
  count       = 2
  name        = "webserver${count.index + 1}"
  hostname    = "webserver${count.index + 1}"
  platform_id = "standard-v3"
  zone        = "ru-central1-${count.index == 0? "a" : "b"}"

  resources {
    cores         = 2
    memory        = 2
    core_fraction = 20
  }
  
  scheduling_policy {
    preemptible = true
  }

  boot_disk {
    initialize_params {
      image_id = "fd8s4a9mnca2bmgol2r8"
      size     = 10
      type     = "network-hdd"
    }
  }

  network_interface {
    subnet_id          = count.index == 0? yandex_vpc_subnet.web-sub-a.id : yandex_vpc_subnet.web-sub-b.id  
    security_group_ids = [ yandex_vpc_security_group.webserver-sg.id, yandex_vpc_security_group.internet-sg.id ]    
  }

  metadata = {
    user-data = "${file("/home/s_yaremko/devops/meta.yml")}"
  }
}

#bastion
resource "yandex_compute_instance" "bastion" {
  name        = "bastion"
  hostname    = "bastion"
  platform_id = "standard-v3"
  zone        = "ru-central1-a"

  resources {
    cores         = 2
    memory        = 2
    core_fraction = 20
  }
  
  scheduling_policy {
    preemptible = true
  }

  boot_disk {
    initialize_params {
      image_id = "fd806u1okplml22f4pmo"
      size     = 10
      type     = "network-hdd"
    }
  }

  network_interface {    
    subnet_id          =  yandex_vpc_subnet.external-sub-c.id     
    security_group_ids = [ yandex_vpc_security_group.bastion-sg.id, yandex_vpc_security_group.internet-sg.id ]  
    nat                = true  
  }
  
  metadata = {
    user-data = "${file("/home/s_yaremko/devops/meta_bastion.yml")}"
  }
}

output "bastion_nat_ip_address" {
  value = yandex_compute_instance.bastion.network_interface.0.nat_ip_address
}

#elasticsearch
resource "yandex_compute_instance" "elastic" {
  name        = "elastic"
  hostname    = "elastic"
  platform_id = "standard-v3"
  zone        = "ru-central1-a"

  resources {
    cores         = 4
    memory        = 4
    core_fraction = 20
  }
  
  scheduling_policy {
    preemptible = true
  }

  boot_disk {
    initialize_params {
      image_id = "fd8s4a9mnca2bmgol2r8"
      size     = 15
      type     = "network-hdd"
    }
  }

  network_interface {    
    subnet_id          =  yandex_vpc_subnet.external-sub-c.id     
    security_group_ids = [ yandex_vpc_security_group.elastic-sg.id, yandex_vpc_security_group.internet-sg.id ]      
  }
  
  metadata = {
    user-data = "${file("/home/s_yaremko/devops/meta.yml")}"
  }
}

#kibana
resource "yandex_compute_instance" "kibana" {
  name        = "kibana"
  hostname    = "kibana"
  platform_id = "standard-v3"
  zone        = "ru-central1-a"

  resources {
    cores         = 2
    memory        = 4
    core_fraction = 20
  }
  
  scheduling_policy {
    preemptible = true
  }

  boot_disk {
    initialize_params {
      image_id = "fd8s4a9mnca2bmgol2r8"
      size     = 15
      type     = "network-hdd"
    }
  }

  network_interface {    
    subnet_id          = yandex_vpc_subnet.external-sub-c.id    
    security_group_ids = [ yandex_vpc_security_group.kibana-sg.id, yandex_vpc_security_group.internet-sg.id ]  
    nat = true    
  }
  
  metadata = {
    user-data = "${file("/home/s_yaremko/devops/meta.yml")}"
  }
}

output "kibana_nat_ip_address" {
  value = yandex_compute_instance.kibana.network_interface.0.nat_ip_address
}

#zabbix_server
resource "yandex_compute_instance" "zabbix" {
  name        = "zabbix"
  hostname    = "zabbix"
  platform_id = "standard-v3"
  zone        = "ru-central1-a"

  resources {
    cores         = 2
    memory        = 2
    core_fraction = 20
  }
  
  scheduling_policy {
    preemptible = true
  }

  boot_disk {
    initialize_params {
      image_id = "fd8s4a9mnca2bmgol2r8"
      size     = 20
      type     = "network-hdd"
    }
  }

  network_interface {    
    subnet_id          = yandex_vpc_subnet.external-sub-c.id    
    security_group_ids = [ yandex_vpc_security_group.zabbix-sg.id, yandex_vpc_security_group.internet-sg.id ]  
    nat = true    
  }
  
  metadata = {
    user-data = "${file("/home/s_yaremko/devops/meta.yml")}"
  }
}

output "zabbix_nat_ip_address" {
  value = yandex_compute_instance.zabbix.network_interface.0.nat_ip_address
}

#network
resource "yandex_vpc_network" "insommnia-net" {
  name = "insommnia-net"
}

#subnets
resource "yandex_vpc_subnet" "web-sub-a" {
  name = "web-sub-a"
  v4_cidr_blocks = ["192.168.1.0/24"]
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.insommnia-net.id
  route_table_id = yandex_vpc_route_table.bastion-route.id
}

resource "yandex_vpc_subnet" "web-sub-b" {
  name = "web-sub-b"
  v4_cidr_blocks = ["192.168.2.0/24"]
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.insommnia-net.id
  route_table_id = yandex_vpc_route_table.bastion-route.id
}

resource "yandex_vpc_subnet" "external-sub-c" {
  name = "external-sub-c"
  v4_cidr_blocks = ["192.168.3.0/24"]
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.insommnia-net.id
}

resource "yandex_vpc_route_table" "bastion-route" {
  name        = "bastion-route"

  depends_on = [ yandex_compute_instance.bastion ]

  network_id = yandex_vpc_network.insommnia-net.id

  static_route {
    destination_prefix = "0.0.0.0/0"
    next_hop_address   = yandex_compute_instance.bastion.network_interface.0.ip_address
  }
}

#security_ for internet
resource "yandex_vpc_security_group" "internet-sg" {
  name        = "internet-sg"
  network_id  = yandex_vpc_network.insommnia-net.id

  egress {
    protocol       = "ANY"    
    v4_cidr_blocks = ["0.0.0.0/0"] 
    from_port      = 0
    to_port        = 65535 
  }

  ingress {
    protocol       = "ICMP"    
    v4_cidr_blocks = ["0.0.0.0/0"] 
  }
}

#security_group for bastion
resource "yandex_vpc_security_group" "bastion-sg" {
  name        = "bastion-security-group"
  network_id  = yandex_vpc_network.insommnia-net.id

  ingress {
    protocol          = "ANY"
    from_port         = 0
    to_port           = 65535
    v4_cidr_blocks = ["192.168.1.0/24", "192.168.2.0/24"]  
  }

  ingress {
    protocol       = "TCP"
    v4_cidr_blocks = ["0.0.0.0/0"]        
    port           = 22
   }  
}

#security_group for alb
resource "yandex_vpc_security_group" "alb-sg" {
  name        = "alb_load_balancer-security-group"
  network_id  = yandex_vpc_network.insommnia-net.id

  ingress {
    protocol       = "TCP"   
    v4_cidr_blocks = ["0.0.0.0/0"]    
    port           = 80
   }

   ingress {
    protocol       = "TCP"   
    v4_cidr_blocks = ["0.0.0.0/0"]    
    port           = 443
   }

   ingress {
    protocol       = "TCP"   
    predefined_target = "loadbalancer_healthchecks"        
    port           = 30080     
   }
}

#security_group for webserver
resource "yandex_vpc_security_group" "webserver-sg" {
  name        = "webserver-security-group"
  network_id  = yandex_vpc_network.insommnia-net.id
  
  ingress {
    protocol       = "TCP"    
    security_group_id = yandex_vpc_security_group.alb-sg.id
  }

  ingress {
    protocol          = "TCP"      
    security_group_id = yandex_vpc_security_group.bastion-sg.id   
    port              = 22
   }    

  ingress {
    protocol       = "TCP"    
    security_group_id = yandex_vpc_security_group.zabbix-sg.id   
    from_port         = 10050
    to_port           = 10051
  }
}

#security_group for elasticsearch
resource "yandex_vpc_security_group" "elastic-sg" {
  name        = "elastic-security-group"
  network_id  = yandex_vpc_network.insommnia-net.id
  
  ingress {
    protocol       = "TCP"    
    v4_cidr_blocks = ["0.0.0.0/0"]  
    port           = 9200
  }

  ingress {
    protocol          = "TCP"      
    security_group_id = yandex_vpc_security_group.bastion-sg.id   
    port              = 22
   }    
}

#security_group for kibana
resource "yandex_vpc_security_group" "kibana-sg" {
  name        = "kibana-security-group"
  network_id  = yandex_vpc_network.insommnia-net.id
  
  ingress {
    protocol       = "TCP"    
    v4_cidr_blocks = ["0.0.0.0/0"]  
    port           = 5601
  }

  ingress {
    protocol          = "TCP"      
    security_group_id = yandex_vpc_security_group.bastion-sg.id   
    port              = 22
  }    
}

#security_group for zabbix
resource "yandex_vpc_security_group" "zabbix-sg" {
  name        = "zabbix-security-group"
  network_id  = yandex_vpc_network.insommnia-net.id
  
  ingress {
    protocol       = "TCP"    
    v4_cidr_blocks = ["0.0.0.0/0"]  
    from_port         = 10050
    to_port           = 10051
  }

  ingress {
    protocol       = "TCP"    
    v4_cidr_blocks = ["0.0.0.0/0"]  
    port         = 80
  }

  ingress {
    protocol          = "TCP"      
    security_group_id = yandex_vpc_security_group.bastion-sg.id   
    port              = 22
  }    
}

#target group
resource "yandex_alb_target_group" "insommnia-tg" {
  name      = "insommnia-tg"

  target {
    subnet_id  = yandex_vpc_subnet.web-sub-a.id
    ip_address = yandex_compute_instance.webserver[0].network_interface.0.ip_address
  }

  target {
    subnet_id  = yandex_vpc_subnet.web-sub-b.id
    ip_address = yandex_compute_instance.webserver[1].network_interface.0.ip_address    
  }
}

#backend group
resource "yandex_alb_backend_group" "insommnia-bg" {
  name      = "insommnia-bg"

  http_backend {
    name = "insommnia-http"
    port = 80
  target_group_ids = [yandex_alb_target_group.insommnia-tg.id]
    healthcheck {
      timeout = "10s"
      interval = "2s"
      http_healthcheck {
        path  = "/"
      }
    }
  }
}

#http-router
resource "yandex_alb_http_router" "insommnia-rt" {
  name      = "insommnia-rt"
}

#virtual host
resource "yandex_alb_virtual_host" "insommnia-vh" {
  name      = "insommnia-vh"
  http_router_id = yandex_alb_http_router.insommnia-rt.id
  route {
    name = "insommnia-route"
    http_route {
      http_route_action {
        backend_group_id = yandex_alb_backend_group.insommnia-bg.id
      }
    }
  }
}

#load-balancer
resource "yandex_alb_load_balancer" "insommnia-lb" {
  name = "insommnia-lb"

  network_id  = yandex_vpc_network.insommnia-net.id
  security_group_ids = [ yandex_vpc_security_group.alb-sg.id, yandex_vpc_security_group.internet-sg.id ]

  allocation_policy {
    location {
      zone_id   = "ru-central1-a"
      subnet_id = yandex_vpc_subnet.web-sub-a.id
    }
    location {
      zone_id   = "ru-central1-b"
      subnet_id = yandex_vpc_subnet.web-sub-b.id
    }
  }

  listener {
    name = "insommnia-list"
    endpoint {
      address {
        external_ipv4_address {
        }
      }
      ports = [ 80 ]
    }
    http {
      handler {
        http_router_id = yandex_alb_http_router.insommnia-rt.id
      }
    }
  }
}

output "alb_external_ip_address" {
  value = yandex_alb_load_balancer.insommnia-lb.listener.0.endpoint.0.address.0.external_ipv4_address[0].address
}
```
