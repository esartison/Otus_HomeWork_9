# Домашнее задание Сартисона Евгения №9

## **(1) Развернуть managed PostgreSQL в двух облаках (на выбор: VK Cloud, Yandex Cloud, SberCloud).##
Минимальные параметры: 1 vCPU, 1 ГБ RAM

Для сравнения выбрал 2 облака: VK Cloud и  Yandex Cloud

**VK Cloud**
выбрал 2 CPU, 8Gb RAM и SSD
<img width="1240" height="587" alt="image" src="https://github.com/user-attachments/assets/98b4fe84-c175-4a46-bdcb-3dc73d99d723" />


**Yandex Cloud**

выбрал 2 CPU, 8Gb RAM и HDD

<img width="662" height="618" alt="image" src="https://github.com/user-attachments/assets/baa13d6a-d559-4dd1-8ecf-0a1d83a50c94" />



    

## **(2)Настроить доступ с вашего IP ##
Проверить подключение через psql

**VK Cloud**
Зашел с самой виртуальной машины в базу
<img width="874" height="149" alt="image" src="https://github.com/user-attachments/assets/fdfb9ecc-2bfc-45d4-a6f6-2dfe7a8b1858" />

Попробовал подключиться к базе данных с management ноды otusmgt01(виртуальная машина в VK Cloud) - доступ был без дополнительных настроек. 
<img width="835" height="137" alt="image" src="https://github.com/user-attachments/assets/ca9b1ff5-b75f-4f12-81f2-9c342353d78c" />

Внешний доступ к базе запретил при создании.

При создании в группе безопасности разрешил подключения по ssh. Тоесть, никаких дополнительных действий не предпринемал, все и так работало. 


**Yandex Cloud**

Нужен доступ к базе данных с management ноды otusmgt01(виртуальная машина Yandex Cloud)

(a) Создал группу безопасности esartisonconn

<img width="626" height="235" alt="image" src="https://github.com/user-attachments/assets/3a218054-b78e-4b1a-bde7-73070ac20094" />


(б) добавил правило, чтобы мог подключаться с otusmgt01

Для генерации CIDR использовал сайт [IP Range To CIDR](https://www.ipaddressguide.com/cidr)

(в) Добавление сертификата на локальную машину
```
esartison@otusmgt01:~$  wget "https://storage.yandexcloud.net/cloud-certs/CA.pem" \
>     --output-document ~/.postgresql/root.crt && \
> chmod 0600 ~/.postgresql/root.crt
--2025-07-27 17:08:30--  https://storage.yandexcloud.net/cloud-certs/CA.pem
Resolving storage.yandexcloud.net (storage.yandexcloud.net)... 213.180.193.243, 2a02:6b8::1d9
Connecting to storage.yandexcloud.net (storage.yandexcloud.net)|213.180.193.243|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3579 (3.5K) [application/x-x509-ca-cert]
Saving to: ‘/home/esartison/.postgresql/root.crt’

/home/esartison/.postgresql/root.crt                        100%[========================================================================================================================================>]   3.50K  --.-KB/s    in 0s

2025-07-27 17:08:30 (1.06 GB/s) - ‘/home/esartison/.postgresql/root.crt’ saved [3579/3579]
```

(д) проверка подключения
```
esartison@otusmgt01:~$ psql "host=rc1a-i97ee0q54nlm3sj0.mdb.yandexcloud.net,rc1d-605o1bcrlkt8sopi.mdb.yandexcloud.net \
    port=6432 \
    sslmode=verify-full \
    dbname=pgdb \
    user=pguser \
    target_session_attrs=read-write"
Password for user pguser:
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1), server 17.5 (Ubuntu 17.5-201-yandex.59510.7fea32f73d))
pgdb=>
pgdb=> SELECT current_database();
 current_database
------------------
 pgdb
(1 row)

pgdb=> SELECT current_catalog;
 current_catalog
-----------------
 pgdb
(1 row)
```
Все прошло успешно!

## **(3) Сравнить характеристики: ##
**- Стоимость в месяц (включая резервные копии)**

**VK Cloud**

**Yandex Cloud**

| Сущность | Yandex Cloud  | VK Cloud  |
|---|---|---|
|кол-во узлов   |2   |2   |
|CPU   |2   |   |
|RAM   |8Gb   | 0.01s  |
|HDD   | 10Gb  | 10Gb  |
|глубина хранения бэкапов  | 7 дней  | 0.04s  |
|Стоимость в месяц   | 10 613  | 0.04s  |
|pgbench latency   |  134.903 ms  | 0.04s  |



**- Latency при выполнении запросов**
Для проверки latency используем pgbench

**VK Cloud**

**Yandex Cloud**
Выполняем pgbech со следующими аргументами: pgbench -c <num_clients> -T <time_limit_seconds> -h <remote_host> -p <remote_port> -U <username> <database_name>
```
esartison@otusmgt01:~$ pgbench -c 5 -T 300 -h rc1a-i97ee0q54nlm3sj0.mdb.yandexcloud.net -p 6432 -U pguser pgdb
Password:
pgbench (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1), server 17.5 (Ubuntu 17.5-201-yandex.59510.7fea32f73d))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 5
number of threads: 1
maximum number of tries: 1
duration: 300 s
number of transactions actually processed: 11115
number of failed transactions: 0 (0.000%)
latency average = 134.903 ms
initial connection time = 205.872 ms
tps = 37.063703 (without initial connection time)
```
latency average = 134.903 ms




**- Удобство управления (интерфейс, документация)**

**VK Cloud**
Понравилось что
- диск может расширяться автоматически
- тонкая настройка бэкапов включая возможность выбора полного востановления или PITR
- более щепетильное отношение к безопасности, не дает возм-ти использовать простой пароль
- более низкая цена
- нет необходимости привязывать банковскую карту
- возможность подключится по ssh к серверу

Не Понравилось
- Нельзя выбрать технологическое окно
- отсутсвие расширенных настроек базы данных
- менее интуитивно понятный интерфейс
  
  
**Yandex Cloud**
Понравилось что
- диск может расширяться автоматически
- Можно выбрать технологическое окно
- возможность расширенных настроек
- низкий порог входа
- хорошая документация

Не Понравилось
- настройка удаленного доступа требует сноровки и не все интуитивно понятно
- Latency в графиках не показывается, только с помощью pgbench можно настроить.
- высокая цена


## **(4) Документировать процесс: ##
Какие облака выбрали и почему

-- Выбрал **VK Cloud** и **Yandex Cloud** для тестирования потому-что они являются лидерами на рынке, хотел понять приемущества обоих. 
СберКлауд в тройку не входит. 

Нашел пару статей, которые заслуживают некоторое доверие: [Мы протестировали разные облака в России на скорость PostgreSQL](https://habr.com/ru/companies/h3llo_cloud/articles/894914/) и 
[Сравнение отечественных облачных сервисов](https://www.arsis.ru/blog/sravneniye-otechestvennykh-oblachnykh-servisov)

По итогам анализа, отдаю свое предпочтение 


## **(5) С какими проблемами столкнулись (например, сложность настройки доступа) ##
Вывод: какое облако лучше подходит для BananaFlow?


**VK Cloud**

**Yandex Cloud**
Все интуитивно понятно, в целом нареканий нет. Работать можно. 




★ Задание со звездочкой ★:

Автоматизировать развертывание с помощью Terraform & Ansible:

**Terraform для создания кластеров**

Создал отдельную management машину для запуска Terraform и Ansible
<img width="1251" height="107" alt="image" src="https://github.com/user-attachments/assets/9e8cc786-66e5-4313-a657-5cfe8413bf26" />

Reference: https://yandex.cloud/en/docs/managed-postgresql/tf-ref 
https://gitops.ru/tools/clouds/yandex/tools/terraform/
https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#cli_2



config
```
esartison@otusmgt01:~$ cat yandex_cloud_create_managed_PG_cluster.tf
//
// Create a new MDB PostgreSQL Cluster.
//
resource "yandex_mdb_postgresql_cluster" "my_cluster" {
  name        = "esartisoncl"
  environment = "PRESTABLE"
  network_id  = yandex_vpc_network.default-esartison-network.id

  config {
    version = 15
    resources {
      resource_preset_id = "s2.micro"
      disk_type_id       = "network-ssd"
      disk_size          = 16
    }
    postgresql_config = {
      max_connections                = 395
      enable_parallel_hash           = true
      autovacuum_vacuum_scale_factor = 0.34
      default_transaction_isolation  = "TRANSACTION_ISOLATION_READ_COMMITTED"
      shared_preload_libraries       = "SHARED_PRELOAD_LIBRARIES_AUTO_EXPLAIN,SHARED_PRELOAD_LIBRARIES_PG_HINT_PLAN"
    }
  }

  maintenance_window {
    type = "WEEKLY"
    day  = "SAT"
    hour = 12
  }

  host {
    zone      = "ru-central1-d"
    subnet_id = yandex_vpc_subnet.default-esartison-network.id
  }
}

// Auxiliary resources
resource "yandex_vpc_network" "default-esartison-network" {}

resource "yandex_vpc_subnet" "default-esartison-network" {
  zone           = "ru-central1-d"
  network_id     = yandex_vpc_network.default-esartison-network.id
  v4_cidr_blocks = ["10.5.0.0/24"]
}

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  token  =  "y0_*****EpRL2g"
  folder_id = "b1gvmo2ucr841odj1o30"
  zone      = "ru-central1-a"
}



resource "yandex_mdb_postgresql_database" "my_db" {
  cluster_id = yandex_mdb_postgresql_cluster.my_cluster.id
  name       = "esartisondb"
  owner      = yandex_mdb_postgresql_user.my_user.name
  lc_collate = "en_US.UTF-8"
  lc_type    = "en_US.UTF-8"
  extension {
    name = "uuid-ossp"
  }
  extension {
    name = "xml2"
  }
}


resource "yandex_mdb_postgresql_user" "my_user" {
  cluster_id = yandex_mdb_postgresql_cluster.my_cluster.id
  name       = "esartison"
  password   = "password"
  conn_limit = 50
  settings = {
    default_transaction_isolation = "read committed"
    log_min_duration_statement    = 5000
  }
}

```

Установка
```
esartison@otusmgt01:~$ terraform apply
.......
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_mdb_postgresql_cluster.my_cluster: Creating...
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [00m10s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [00m20s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [00m30s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [00m40s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [00m50s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [01m00s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [01m10s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [01m20s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [01m30s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [01m40s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [01m50s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [02m00s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [02m10s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [02m20s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [02m30s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [02m40s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [02m50s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [03m00s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [03m10s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [03m20s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [03m30s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [03m40s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [03m50s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [04m00s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [04m10s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [04m20s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [04m30s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [04m40s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [04m50s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [05m00s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [05m10s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [05m20s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [05m30s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [05m40s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [05m50s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [06m00s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [06m10s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Still creating... [06m20s elapsed]
yandex_mdb_postgresql_cluster.my_cluster: Creation complete after 6m28s [id=c9q98ftudf56ef28en1p]
yandex_mdb_postgresql_user.my_user: Creating...
yandex_mdb_postgresql_user.my_user: Still creating... [00m10s elapsed]
yandex_mdb_postgresql_user.my_user: Still creating... [00m20s elapsed]
yandex_mdb_postgresql_user.my_user: Still creating... [00m30s elapsed]
yandex_mdb_postgresql_user.my_user: Creation complete after 40s [id=c9q98ftudf56ef28en1p:esartison]
yandex_mdb_postgresql_database.my_db: Creating...
yandex_mdb_postgresql_database.my_db: Still creating... [00m10s elapsed]
yandex_mdb_postgresql_database.my_db: Still creating... [00m20s elapsed]
yandex_mdb_postgresql_database.my_db: Still creating... [00m30s elapsed]
yandex_mdb_postgresql_database.my_db: Creation complete after 37s [id=c9q98ftudf56ef28en1p:esartisondb]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
esartison@otusmgt01:~$
```

Проверка
<img width="633" height="474" alt="image" src="https://github.com/user-attachments/assets/c6904886-70a8-442e-a500-184b585e1641" />
<img width="1101" height="125" alt="image" src="https://github.com/user-attachments/assets/06a28c7a-8aa3-4553-b743-f056191b37c6" />
<img width="953" height="138" alt="image" src="https://github.com/user-attachments/assets/10a338b4-7c4f-4316-8149-e49024f55a75" />

Все прошло успешно и создалось как и планировали. 



**Ansible для настройки подключения и тестовых запросов**

Не хватило времени чтобы хорошо потестировать и описать. 

Есть playbook для этих целей: https://ansible.readthedocs.io/projects/ansible/2.10/collections/community/general/postgresql_ping_module.html



Формат сдачи:
GitHub-репозиторий с:
README.md (описание шагов, сравнение, выводы)
Конфиги Terraform/Ansible (если делали бонус)
Скриншоты работающих кластеров
