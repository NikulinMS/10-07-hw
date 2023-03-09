# Домашнее задание к занятию "`Отказоустойчивость в облаке`" - `Никулин Михаил Сергеевич`



---

### Задание 1


```
root@nikulin:~/new# cat main.tf
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}


provider "yandex" {
  token = "y0_AgAAAABEoWjbAATuwQAAAADS4wxXvWtjMg8PS4KRoTr1PIANTWqz8R4"
  cloud_id = "b1gtheioau4s71j2mu0u"
  folder_id = "b1gtheioau4s71j2mu0u"
  zone = "ru-central1-b"
}


resource "yandex_compute_instance" "vm" {

count = 2
name = "vm${count.index}"

  resources {
    cores  = 2
    memory = 2
    core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id = "fd8pqqqelpfjceogov30"
      size = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }

  metadata = {
    user-data = "${file("./meta.txt")}"

  }

}

resource "yandex_vpc_network" "network-1" {
  name = "network1"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet1"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}

resource "yandex_lb_target_group" "netol-1" {
  name      = "netol-1"

  target {
    subnet_id = "${yandex_vpc_subnet.subnet-1.id}"
    address   = "${yandex_compute_instance.vm[0].network_interface.0.ip_address}"
  }

  target {
    subnet_id = "${yandex_vpc_subnet.subnet-1.id}"
    address   = "${yandex_compute_instance.vm[1].network_interface.0.ip_address}"
  }

}

resource "yandex_lb_network_load_balancer" "balancer-1" {
  name = "balancer1"
  listener {
    name = "my-listener"
    port = 80
  }
  attached_target_group {
    target_group_id = "${yandex_lb_target_group.netol-1.id}"
    healthcheck {
      name = "http"
        http_options {
          port = 80
          path = "/"
        }
     }
  }
}

```

![task_1_1.png](img%2Ftask_1_1.png)
![task_1_2.png](img%2Ftask_1_2.png)


---



---
## Дополнительные задания (со звездочкой*)


### Задание 2*


```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}


provider "yandex" {
  token     = "y0_AgAAAABEoWjbAATuwQAAAADS4wxXvWtjMg8PS4KRoTr1PIANTWqz8R4"
  cloud_id  = "b1gtheioau4s71j2mu0u"
  folder_id = "b1gtheioau4s71j2mu0u"
  zone      = "ru-central1-b"
}


resource "yandex_compute_instance_group" "group1" {
  name                = "test-ig"
  folder_id           = "b1gtheioau4s71j2mu0u"
  service_account_id  = "${yandex_iam_service_account.ig-sa.id}"
  instance_template {
    platform_id = "standard-v1"
    resources {
      memory = 2
      cores  = 2
      core_fraction = 20
    }
    boot_disk {
      mode = "READ_WRITE"
      initialize_params {
        image_id = "fd8a67rb91j689dqp60h"
        size     = 4
      }
    }
    network_interface {
      network_id = yandex_vpc_network.network-1.id
      subnet_ids = ["${yandex_vpc_subnet.subnet-1.id}"]
      nat        = true
    }

    metadata = {
      ssh-keys = "debian:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCS7zgkhHyRIZF+A2lEWwY2dwlUyWo1XEVB6irNbmrXnH5JRw05RKVLtodZ1ooIeRyKuvqgrAo7smoBjq5bZZAmi8EaNn+4NP+wBAN4pJjtt9SSqUb+GGBd3l5KGzn5GQhiFo48bch8xnc+B5Nl8KYwvUHCtkD3IpllbIWsNlhHGiTvMjoCt8Mr5eNAoobQGuddH/ojGxYViUf/qqVlq3y/sKaoWhO9mlEFOh+/0oW9sHHeKWDNnOuAORIx4c8igU3wdocs9gfWgFeExFq8QIk3BoN885vSE1HutQOnmuCOAXVS1OTN75hS+A249kiiDhT8IVzgP1xwS+8hO/NQ6Zgh rsa-key-20230308"
      user-data = "${file("./metadata.yaml")}"
    }
  }

  load_balancer {
    target_group_name        = "target-group"
    target_group_description = "load balancer target group"
  }


  scale_policy {
    fixed_scale {
      size = 2
    }
  }

  allocation_policy {
    zones = ["ru-central1-b"]
  }

  deploy_policy {
    max_unavailable = 2
    max_creating    = 2
    max_expansion   = 2
    max_deleting    = 2
  }
}
resource "yandex_iam_service_account" "ig-sa" {
  name        = "ig-sa"
  description = "service account to manage IG"
}

resource "yandex_resourcemanager_folder_iam_binding" "editor" {
  folder_id = "b1gtheioau4s71j2mu0u"
  role      = "editor"
  members = [
    "serviceAccount:${yandex_iam_service_account.ig-sa.id}",
  ]
}


resource "yandex_vpc_network" "network-1" {
  name = "network1"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet1"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}


resource "yandex_lb_network_load_balancer" "balancer-1" {
  name = "balancer1"
  listener {
    name = "my-listener"
    port = 80
  }
  attached_target_group {
    target_group_id = "${yandex_compute_instance_group.group1.load_balancer.0.target_group_id}"
    healthcheck {
      name = "http"
        http_options {
          port = 80
          path = "/"
        }
     }
  }
}

```

![task_2_1.png](img%2Ftask_2_1.png)
![task_2_2.png](img%2Ftask_2_2.png)

[terraform.zip](data%2Fterraform.zip)