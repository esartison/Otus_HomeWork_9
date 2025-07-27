# Домашнее задание Сартисона Евгения №9

## **(1) Развернуть managed PostgreSQL в двух облаках (на выбор: VK Cloud, Yandex Cloud, SberCloud).##
Минимальные параметры: 1 vCPU, 1 ГБ RAM

Для сравнения выбрал 2 облака: VK Cloud и  Yandex Cloud

**VK Cloud**

**Yandex Cloud**



## **(2)Настроить доступ с вашего IP ##
Проверить подключение через psql

**VK Cloud**

**Yandex Cloud**

## **(3) Сравнить характеристики: ##
Стоимость в месяц (включая резервные копии)

**VK Cloud**

**Yandex Cloud**

Latency при выполнении запросов


**VK Cloud**

**Yandex Cloud**

Удобство управления (интерфейс, документация)

**VK Cloud**

**Yandex Cloud**


## **(4) Документировать процесс: ##
Какие облака выбрали и почему


## **(5) С какими проблемами столкнулись (например, сложность настройки доступа) ##
Вывод: какое облако лучше подходит для BananaFlow?


**VK Cloud**

**Yandex Cloud**












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



Формат сдачи:
GitHub-репозиторий с:
README.md (описание шагов, сравнение, выводы)
Конфиги Terraform/Ansible (если делали бонус)
Скриншоты работающих кластеров

Алексей представляет отчет Олегу:

— "VK Cloud дешевле на 15%, но SberCloud дает меньшую задержку для наших филиалов в Азии. И главное — никаких ночных дежурств!"

Олег одобрительно кивает:

— "Значит, в Европу ставим SberCloud, а для России — VK. И чтобы всё автоматизировать!"


Критерии оценки:
✅ Кластеры развернуты в 2 облаках
✅ Проведено сравнение (цена, latency, удобство)
✅ Документация с выводами

Бонусные баллы:
★ Полная автоматизация (Terraform + Ansible)
★ Тестирование под нагрузкой (например, pgbench)
