# savilov-devops
# Дипломный практикум в Yandex.Cloud
  * [Цели:](#цели)
  * [Этапы выполнения:](#этапы-выполнения)
     * [Создание облачной инфраструктуры](#создание-облачной-инфраструктуры)
     * [Создание Kubernetes кластера](#создание-kubernetes-кластера)
     * [Создание тестового приложения](#создание-тестового-приложения)
     * [Подготовка cистемы мониторинга и деплой приложения](#подготовка-cистемы-мониторинга-и-деплой-приложения)
     * [Установка и настройка CI/CD](#установка-и-настройка-cicd)
  * [Что необходимо для сдачи задания?](#что-необходимо-для-сдачи-задания)
  * [Как правильно задавать вопросы дипломному руководителю?](#как-правильно-задавать-вопросы-дипломному-руководителю)

**Перед началом работы над дипломным заданием изучите [Инструкция по экономии облачных ресурсов](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD).**

---
## Цели:

1. Подготовить облачную инфраструктуру на базе облачного провайдера Яндекс.Облако.
2. Запустить и сконфигурировать Kubernetes кластер.
3. Установить и настроить систему мониторинга.
4. Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. Настроить CI для автоматической сборки и тестирования.
6. Настроить CD для автоматического развёртывания приложения.

---
## Этапы выполнения:


### Создание облачной инфраструктуры

Для начала необходимо подготовить облачную инфраструктуру в ЯО при помощи [Terraform](https://www.terraform.io/).

Особенности выполнения:

- Бюджет купона ограничен, что следует иметь в виду при проектировании инфраструктуры и использовании ресурсов;
Для облачного k8s используйте региональный мастер(неотказоустойчивый). Для self-hosted k8s минимизируйте ресурсы ВМ и долю ЦПУ. В обоих вариантах используйте прерываемые ВМ для worker nodes.

Предварительная подготовка к установке и запуску Kubernetes кластера.

1. Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя
2. Подготовьте [backend](https://www.terraform.io/docs/language/settings/backends/index.html) для Terraform:  
   а. Рекомендуемый вариант: S3 bucket в созданном ЯО аккаунте(создание бакета через TF)
   б. Альтернативный вариант:  [Terraform Cloud](https://app.terraform.io/)
3. Создайте конфигурацию Terrafrom, используя созданный бакет ранее как бекенд для хранения стейт файла. Конфигурации Terraform для создания сервисного аккаунта и бакета и основной инфраструктуры следует сохранить в разных папках.
4. Создайте VPC с подсетями в разных зонах доступности.
5. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий.
6. В случае использования [Terraform Cloud](https://app.terraform.io/) в качестве [backend](https://www.terraform.io/docs/language/settings/backends/index.html) убедитесь, что применение изменений успешно проходит, используя web-интерфейс Terraform cloud.

Ожидаемые результаты:

1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий, стейт основной конфигурации сохраняется в бакете или Terraform Cloud
2. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

---
### Создание Kubernetes кластера

На этом этапе необходимо создать [Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфраструктуры.   Требуется обеспечить доступ к ресурсам из Интернета.

Это можно сделать двумя способами:

1. Рекомендуемый вариант: самостоятельная установка Kubernetes кластера.  
   а. При помощи Terraform подготовить как минимум 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера. Тип виртуальной машины следует выбрать самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете, что необходимо сменить тип инстанса, используйте Terraform для внесения изменений.  
   б. Подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)  
   в. Задеплоить Kubernetes на подготовленные ранее инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
2. Альтернативный вариант: воспользуйтесь сервисом [Yandex Managed Service for Kubernetes](https://cloud.yandex.ru/services/managed-kubernetes)  
  а. С помощью terraform resource для [kubernetes](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_cluster) создать **региональный** мастер kubernetes с размещением нод в разных 3 подсетях      
  б. С помощью terraform resource для [kubernetes node group](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_node_group)
  
Ожидаемый результат:

1. Работоспособный Kubernetes кластер.
2. В файле `~/.kube/config` находятся данные для доступа к кластеру.
3. Команда `kubectl get pods --all-namespaces` отрабатывает без ошибок.

---
### Создание тестового приложения

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:

1. Рекомендуемый вариант:  
   а. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.  
   б. Подготовьте Dockerfile для создания образа приложения.  
2. Альтернативный вариант:  
   а. Используйте любой другой код, главное, чтобы был самостоятельно создан Dockerfile.

Ожидаемый результат:

1. Git репозиторий с тестовым приложением и Dockerfile.
2. Регистри с собранным docker image. В качестве регистри может быть DockerHub или [Yandex Container Registry](https://cloud.yandex.ru/services/container-registry), созданный также с помощью terraform.

---
### Подготовка cистемы мониторинга и деплой приложения

Уже должны быть готовы конфигурации для автоматического создания облачной инфраструктуры и поднятия Kubernetes кластера.  
Теперь необходимо подготовить конфигурационные файлы для настройки нашего Kubernetes кластера.

Цель:
1. Задеплоить в кластер [prometheus](https://prometheus.io/), [grafana](https://grafana.com/), [alertmanager](https://github.com/prometheus/alertmanager), [экспортер](https://github.com/prometheus/node_exporter) основных метрик Kubernetes.
2. Задеплоить тестовое приложение, например, [nginx](https://www.nginx.com/) сервер отдающий статическую страницу.

Способ выполнения:
1. Воспользоваться пакетом [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus), который уже включает в себя [Kubernetes оператор](https://operatorhub.io/) для [grafana](https://grafana.com/), [prometheus](https://prometheus.io/), [alertmanager](https://github.com/prometheus/alertmanager) и [node_exporter](https://github.com/prometheus/node_exporter). Альтернативный вариант - использовать набор helm чартов от [bitnami](https://github.com/bitnami/charts/tree/main/bitnami).

2. Если на первом этапе вы не воспользовались [Terraform Cloud](https://app.terraform.io/), то задеплойте и настройте в кластере [atlantis](https://www.runatlantis.io/) для отслеживания изменений инфраструктуры. Альтернативный вариант 3 задания: вместо Terraform Cloud или atlantis настройте на автоматический запуск и применение конфигурации terraform из вашего git-репозитория в выбранной вами CI-CD системе при любом комите в main ветку. Предоставьте скриншоты работы пайплайна из CI/CD системы.

Ожидаемый результат:
1. Git репозиторий с конфигурационными файлами для настройки Kubernetes.
2. Http доступ на 80 порту к web интерфейсу grafana.
3. Дашборды в grafana отображающие состояние Kubernetes кластера.
4. Http доступ на 80 порту к тестовому приложению.
---
### Установка и настройка CI/CD

Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:

1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
2. Автоматический деплой нового docker образа.

Можно использовать [teamcity](https://www.jetbrains.com/ru-ru/teamcity/), [jenkins](https://www.jenkins.io/), [GitLab CI](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/) или GitHub Actions.

Ожидаемый результат:

1. Интерфейс ci/cd сервиса доступен по http.
2. При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
3. При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистри, а также деплой соответствующего Docker образа в кластер Kubernetes.

---
## Что необходимо для сдачи задания?

1. Репозиторий с конфигурационными файлами Terraform и готовность продемонстрировать создание всех ресурсов с нуля.
2. Пример pull request с комментариями созданными atlantis'ом или снимки экрана из Terraform Cloud или вашего CI-CD-terraform pipeline.
3. Репозиторий с конфигурацией ansible, если был выбран способ создания Kubernetes кластера при помощи ansible.
4. Репозиторий с Dockerfile тестового приложения и ссылка на собранный docker image.
5. Репозиторий с конфигурацией Kubernetes кластера.
6. Ссылка на тестовое приложение и веб интерфейс Grafana с данными доступа.
7. Все репозитории рекомендуется хранить на одном ресурсе (github, gitlab)


## Выполнение дипломного практикума:

Для выполнения работы, будtv использовать уже настроенную рабочую машину с ОС Ubuntu-22.04 на WSL, со следующими компонентами:

Terraform v1.10.2

Ansible 2.10.8

Python 3.10.2

Docker 27.2.0

Git 2.34.1

Kubectl v1.31.4

Helm v3.16.3

Yandex Cloud CLI 0.140.0

![img1_1_1](https://github.com/user-attachments/assets/1f40e6a3-b98a-4f57-b31a-25d7d5f1aca2)

### Создание облачной инфраструктуры

Создам сервисный аккаунт с необходимыми правами для работы с облачной инфраструктурой, подготовим backend для Terraform, использовать будем S3-bucket:
#### main.tf
```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  backend "s3" {
    endpoints = {
      s3 = "https://storage.yandexcloud.net"
    }
    bucket = "savilovvv"
    region = "ru-central1"
    key    = "my-tf-state.tfstate"

       skip_region_validation      = true
       skip_credentials_validation = true
       skip_requesting_account_id  = true # Необходимая опция Terraform для версии 1.6.1 и старше.
       skip_s3_checksum            = true # Необходимая опция при описании бэкенда для Terraform версии 1.6.3 и старше.
  }
}

provider "yandex" {
  token     = var.token
  cloud_id  = var.cloud_id
  folder_id = var.folder_id
  zone = "ru-central-a"
}
```

#### account.tf
```
# Создаем сервисный аккаунт для Terraform
resource "yandex_iam_service_account" "service" {
  folder_id = var.folder_id
  name      = var.account_name
}

# Выдаем роль editor сервисному аккаунту Terraform
resource "yandex_resourcemanager_folder_iam_member" "service_editor" {
  folder_id = var.folder_id
  role      = "editor"
  member    = "serviceAccount:${yandex_iam_service_account.service.id}"
}

# Создаем статический ключ доступа для сервисного аккаунта
resource "yandex_iam_service_account_static_access_key" "terraform_service_account_key" {
  service_account_id = yandex_iam_service_account.service.id
}

# Используем ключ доступа для создания бакета
resource "yandex_storage_bucket" "tf-bucket" {
  bucket     = "for-state"
  access_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.access_key
  secret_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.secret_key

  anonymous_access_flags {
    read = false
    list = false
  }

  force_destroy = true

provisioner "local-exec" {
  command = "echo export ACCESS_KEY=${yandex_iam_service_account_static_access_key.terraform_service_account_key.access_k>}

provisioner "local-exec" {
  command = "echo export SECRET_KEY=${yandex_iam_service_account_static_access_key.terraform_service_account_key.secret_k>}
}
```
Применяем код:

![img1_2](https://github.com/user-attachments/assets/619d8534-12e7-4b83-a4a0-0d4cc0c37a58)

В результате применения этого кода Terraform был создан сервисный аккаунт с правами для редактирования, статический ключ доступа и S3-bucket. 
Переменные ACCESS_KEY и SECRET_KEY будут записаны в файл backend.tfvars. Сделано так потому, что эти данные являются очень чувствительными и 
не рекомендуется их хранить в облаке. Эти переменные будут в экспортированы в оболочку рабочего окружения.

Проверим, создался ли S3-bucket и сервисный аккаунт:

![img1_3_1](https://github.com/user-attachments/assets/5f319a61-b53d-4ced-a863-03f01fd02783)

Сервисный аккаунт и S3-bucket созданы.

После создания S3-bucket также была выполнена настройка для использования его в качестве backend для Terraform. Для этого добавим в main.tf следующий код:
```
backend "s3" {
    endpoints = {
      s3 = "https://storage.yandexcloud.net"
    }
    bucket = "savilovvv"
    region = "ru-central1"
    key    = "my-tf-state.tfstate"

       skip_region_validation      = true
       skip_credentials_validation = true
       skip_requesting_account_id  = true # Необходимая опция Terraform для версии 1.6.1 и старше.
       skip_s3_checksum            = true # Необходимая опция при описании бэкенда для Terraform версии 1.6.3 и старше.
  }
  ```
Этот код настраивает Terraform на использование Yandex Cloud Storage в качестве места для хранения файла состояния terraform.tfstate, 
который содержит информацию о конфигурации и состоянии управляемых Terraform ресурсов. Чтобы код был корректно применен и Terraform 
успешно инициализировался, зададим параметры для доступа к S3 хранилищу. Как писал выше, делать это я буду с помощью переменных окружения:
```
slava@DESKTOP-QKJU13U:~/diplomvvs/terraform/test$ export ACCESS_KEY_ID = YCAJEj62RE-.............
slava@DESKTOP-QKJU13U:~/diplomvvs/terraform/test$ export SECRET_KEY_ID = YCM8MsUMlO-wIChNCxWuiMagrb7............
```
Создам VPC с подсетями в разных зонах доступности:
```
resource "yandex_vpc_network" "diplom" {
  name = var.vpc_name
}
resource "yandex_vpc_subnet" "diplom-subnet1" {
  name           = var.subnet1
  zone           = var.zone1
  network_id     = yandex_vpc_network.diplom.id
  v4_cidr_blocks = var.cidr1
}

resource "yandex_vpc_subnet" "diplom-subnet2" {
  name           = var.subnet2
  zone           = var.zone2
  network_id     = yandex_vpc_network.diplom.id
  v4_cidr_blocks = var.cidr2
}

variable "zone1" {
  type        = string
  default     = "ru-central1-a"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "zone2" {
  type        = string
  default     = "ru-central1-b"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "cidr1" {
  type        = list(string)
  default     = ["10.0.1.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "cidr2" {
  type        = list(string)
  default     = ["10.0.2.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "vpc_name" {
  type        = string
  default     = "diplom"
  description = "VPC network&subnet name"
}

variable "bucket_name" {
  type        = string
  default     = "ft-state"
  description = "VPC network&subnet name"
}

variable "subnet1" {
  type        = string
  default     = "diplom-subnet1"
  description = "subnet name"
}

variable "subnet2" {
  type        = string
  default     = "diplom-subnet2"
  description = "subnet name"
}
```

Описываю код Terraform для создания виртуальных машин для Kubernetes кластера. 
Будем использовать одну Master ноду и две Worker ноды, для экономии ресурсов и бюджета купона.

Инициализирую Terraform:

![img1_5](https://github.com/user-attachments/assets/a1cc15f7-8fd7-4f6b-a0d6-c47c8b89c835)

Terraform успешно инициализирован, backend с типом s3 успешно настроен. Terraform будет использовать этот backend 
для хранения файла состояния terraform.tfstate.

Для проверки правильности кода, можно использовать команды terraform validate и terraform plan. В моём коде ошибок не обнаружено:

![img1_6](https://github.com/user-attachments/assets/68ca647d-629c-4c1e-884a-b8858061fd8b)

Применю код для создания облачной инфраструктуры, состоящей из одной Master ноды, двух Worker нод, сети и подсети:

![img1_7_0](https://github.com/user-attachments/assets/a95f95bb-4f3b-4049-ac91-0d260fd39adb)

Кроме создания сети, подсетей и виртуальных машин, создается ресурс из файла ansible.tf, который по шаблону hosts.tftpl создает inventory файл. 
Этот inventory файл в дальнейшем будет использоваться для развёртывания Kubernetes кластера из репозитория Kubespray.

Также при развёртывании виртуальных машин буду использовать файл cloud-init.yml, который установит на них полезные в дальнейшем пакеты. 
Например, curl, Git, MC, atop и другие.

Код для создания Master ноды находится в файле master.tf - https://github.com/slava1005/savilov-devops/blob/main/terraform/test/master.tf

Код для создания Worker нод находится в файле worker.tf - https://github.com/slava1005/savilov-devops/blob/main/terraform/test/worker.tf

Код для установки необходимых пакетов на виртуальные машины при их развертывании находится 
в файле cloud-init.yml - https://github.com/slava1005/savilov-devops/blob/main/terraform/test/cloud-init.yml

Проверю, создались ли виртуальные машины:

![img1_8](https://github.com/user-attachments/assets/d006e868-ac31-4146-b7b1-c030687c6faf)

Виртуальные машины созданы в разных подсетях и разных зонах доступности.

Также проверю все созданные ресурсы через графический интерфейс:

Сервисный аккаунт:

![img1_9](https://github.com/user-attachments/assets/364423fd-b9eb-405a-b109-c246f40d8bc7)

S3-bucket:

![img1_10](https://github.com/user-attachments/assets/ec96ce80-66f7-4c53-97e6-568541dbb001)

Сеть и подсети:

![img1_11](https://github.com/user-attachments/assets/82236c43-c5f4-4d4f-9542-d9baf6d7bfd0)

Виртуальные машины:

![img1_12](https://github.com/user-attachments/assets/888fbb5c-5d6f-4389-80dd-2f6e27170ba4)

Проверю удаление созданных ресурсов:

![img1_13_14](https://github.com/user-attachments/assets/deb6156c-5f4f-4770-86a8-07cbe4a6e34b)

Созданные виртуальные машины, сеть, подсети, сервисный аккаунт, статический ключ и S3-bucket удаляются успешно.

Полный код Terraform для создания сервисного аккаунта, статического ключа, сети, подсетей, виртуальных машин
и S3-bucket доступен по ссылке: https://github.com/slava1005/savilov-devops/tree/main/terraform/test

### Создание Kubernetes кластера
После развёртывания облачной инфраструктуры, приступаю к развёртыванию Kubernetes кластера.

Разворачивать будем из репозитория Kubespray.

Клонирую репозиторий на свою рабочую машину:

![img1_18](https://github.com/user-attachments/assets/3ddaef47-cc22-4949-adf7-7401662c403c)


При разворачивании облачной инфраструктуры с помощью Terraform применяется следующий код:
```
resource "local_file" "hosts_cfg_kubespray" {
  content  = templatefile("${path.module}/hosts.tftpl", {
    workers = yandex_compute_instance.worker
    masters = yandex_compute_instance.master
  })
  filename = "../../kubespray/inventory/mycluster/hosts.yaml"
}
```
Этот код по пути /test/kubespray/inventory/mycluster/ создаст файл hosts.yaml и по шаблону автоматически заполнит его ip адресами нод.

Сам файл шаблона выглядит следующим образом:
```
all:
  hosts:%{ for idx, master in masters }
    master:
      ansible_host: ${master.network_interface[0].nat_ip_address}
      ip: ${master.network_interface[0].ip_address}
      access_ip: ${master.network_interface[0].nat_ip_address}%{ endfor }%{ for idx, worker in workers }
    worker-${idx + 1}:
      ansible_host: ${worker.network_interface[0].nat_ip_address}
      ip: ${worker.network_interface[0].ip_address}
      access_ip: ${worker.network_interface[0].nat_ip_address}%{ endfor }
  children:
    kube_control_plane:
      hosts:%{ for idx, master in masters }
        ${master.name}:%{ endfor }
    kube_node:
      hosts:%{ for idx, worker in workers }
        ${worker.name}:%{ endfor }
    etcd:
      hosts:%{ for idx, master in masters }
        ${master.name}:%{ endfor }
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```
Перейдем в директорию /test/kubespray/ и запустим установку kubernetes кластера командой
```
ansible-playbook -i inventory/mycluster/hosts.yaml -u ubuntu --become --become-user=root --private-key=~/.ssh/id_ed25519 -e 'ansible_ssh_common_args="-o StrictHostKeyChecking=no"' cluster.yml --flush-cache
```
Спустя довольно продолжительное время установка Kubernetes кластера методом Kubespray завершена:

![img1_19](https://github.com/user-attachments/assets/02d6b9c1-1a5c-4ac1-a7df-f567e6ed2fb3)

Далее нужно создать конфигурационный файл кластера Kubernetes.

Для этого подключаюсь к Master ноде и выполняем следующие команды:

![img1_20](https://github.com/user-attachments/assets/52ccdc0e-f0ca-43ba-b87d-239f38830f20)

Эти команды создают директорию для хранения файла конфигурации, копируют созданный при установке Kubernetes кластера конфигурационный файл 
в созданную директорию и назначает права для пользователя на директорию и файл конфигурации.

Конфигурационный файл создан. Теперь можно проверить доступность подов и нод кластера:

![img1_21](https://github.com/user-attachments/assets/1fa345d8-135f-4245-8e53-1e8ff44e484b)

Поды и ноды кластера доступны и находятся в состоянии готовности, следовательно развёртывание Kubernetes кластера успешно завершено.

### Создание тестового приложения

Создадим отдельный репозиторий для тестового приложения:

![img1_22](https://github.com/user-attachments/assets/a8bab67f-6e85-4c47-8c0f-5ed6497d20f7)

Клонируем репозиторий на свою рабочую машину:

![img1_23](https://github.com/user-attachments/assets/d22e0c38-663e-4954-ba0d-6b0b67719b44)

Создаем статичную страничку, которая будет показывать картинку и текст, делаем коммит и отправляем созданную страницу в репозиторий.

![img1_25](https://github.com/user-attachments/assets/e5985e4e-5354-46d8-b070-627f005dea1e)

Ссылка на репозиторий: https://github.com/slava1005/diplom-test-site

Пишем Dockerfile, который создаст контейнер с nginx и отобразит созданную страницу:
```
FROM nginx:1.27.0
RUN rm -rf /usr/share/nginx/html/*
COPY content/ /usr/share/nginx/html/
EXPOSE 80
```
Авторизуюсь в Docker Hub:

![img1_28](https://github.com/user-attachments/assets/89fe9e0c-2cf4-4d73-ae24-fd11fa7d6a50)

Создадим Docker образ:

![img1_29](https://github.com/user-attachments/assets/94bf2fb1-0c5b-46ae-974a-533a81c54c79)

Проверю, создался ли образ:

![img1_30](https://github.com/user-attachments/assets/c86b08d5-5398-4ffa-831e-d97650cca652)

Образ создан.

Опубликую созданный образ реестре Docker Hub:

![img1_31](https://github.com/user-attachments/assets/37d89706-bf7e-4016-9ab8-6679d7267610)

Проверю наличие образа в реестре Docker Hub:

![img1_32](https://github.com/user-attachments/assets/3f245c58-b1f3-45b2-9170-0b388d527d30)
![img1_32_1](https://github.com/user-attachments/assets/da2e90b1-9fbe-4bb9-8f01-9b1de67de3e0)

Ссылка на реестр Docker Hub: https://hub.docker.com/repository/docker/slava1005/diplom-test-site/general

Образ опубликован, подготовка тестового приложения закончена.

### Подготовка системы мониторинга и деплой приложения

Для удобства управления созданным Kubernetes кластером, скопирую его конфигурационный файл на свою рабочую машину и заменю IP адрес сервера:

![img1_33](https://github.com/user-attachments/assets/fdd13c28-6619-4e39-9b2d-99e4eb8ce2cb)

Проверю результат:

![img1_34](https://github.com/user-attachments/assets/07ed8981-1dc5-43ab-ba34-e01e94133cfa)

Kubernetes кластер доступен с рабочей машины.

Добавлю репозиторий ``` prometheus-community ``` для его установки с помощью helm:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
![img1_35](https://github.com/user-attachments/assets/6bc40161-9378-4fef-a093-daaa4690465a)

Для доступа к Grafana снаружи кластера Kubernetes буду использовать тип сервиса NodePort.

Сохраним значения по умолчанию Helm чарта prometheus-community в файл и отредактируем его:

helm show values prometheus-community/kube-prometheus-stack > helm-prometheus/values.yaml

![img1_36](https://github.com/user-attachments/assets/cf0b1c51-a5ae-4645-a982-12fef8440e59)

Изменю пароль по умолчанию Для входа в Grafana изменим пароль по умолчанию, а также изменим сервис и присвоим ему порт 30050:
```
grafana:
  service:
    portName: http-web
    type: NodePort
    nodePort: 30050
```
Используя Helm и подготовленный файл значений values.yaml выполню установку prometheus-community:

helm upgrade --install monitoring prometheus-community/kube-prometheus-stack --create-namespace -n monitoring -f helm-prometheus/values.yaml

![img1_39](https://github.com/user-attachments/assets/6f9dfea8-ee02-4f7f-8db8-fa790bca1003)

При установке был создан отдельный Namespace с названием monitoring.

Посмотрим результат установки:

![img1_40](https://github.com/user-attachments/assets/01522c7e-12ea-4481-911a-338fd1206560)

Установка была выполнена с заданными в values.yaml значениями.

Файл значений values.yaml, использованный при установке prometheus-community доступен по ссылке: 
https://github.com/slava1005/savilov-devops/blob/main/helm-prometheus/values.yaml

Открою web-интерфейс Grafana и авторизуюсь с заранее заданным в values.yaml паролем::

![img1_42](https://github.com/user-attachments/assets/fdcc1630-f05c-4390-976d-2e967e61725c)

Авторизация проходит успешно, данные о состоянии кластера отображаются на дашбордах:

![img1_43_1](https://github.com/user-attachments/assets/2f417119-7532-44da-96ad-d2fe9f545dd8)

![img1_43](https://github.com/user-attachments/assets/c1b4582d-6c62-41fb-8e7a-7bbfbf7aeb37)

Развёртывание системы мониторинга успешно завершено.

Приступаю к развёртыванию тестового приложения на Kubernetes кластере.

Создаю отдельный Namespace, в котором буду развёртывать тестовое приложение:

![img1_44](https://github.com/user-attachments/assets/93357742-3a83-4486-928b-4c635a035c20)

Пишем манифест Deployment с тестовым приложением:

![img1_45](https://github.com/user-attachments/assets/9e985a1b-b8eb-4fb6-a9f3-dc60b71b0ecd)

Применим манифест Deployment и проверим результат:

![img1_46](https://github.com/user-attachments/assets/ba459fac-3931-42e3-b681-f2bc1d947640)

Deployment создан и запущен. Проверю его работу:

![img1_47](https://github.com/user-attachments/assets/f8c21ce4-37a6-43fe-863d-3c654b97b3fc)

Приложение работает.

Ссылка на манифест Deployment: https://github.com/slava1005/savilov-devops/blob/main/k8s-app/deployment.yaml

Пишем манифест сервиса с типом NodePort для доступа к web-интерфейсу тестового приложения:

![img1_48](https://github.com/user-attachments/assets/1602e291-4ed0-499e-9fb0-7d5d0e6706cc)

Применяю манифест сервиса и проверяю результат:

![img1_49](https://github.com/user-attachments/assets/ffa22c1d-b6a1-4944-9e80-09e6d19bf404)

Сервис создан. Теперь проверю доступ к приложению извне:

![img1_50](https://github.com/user-attachments/assets/581fa44f-2fad-4222-a1fa-43379be3c005)

Сайт открывается, приложение доступно.

Ссылка на манифест сервиса: https://github.com/slava1005/savilov-devops/blob/main/k8s-app/service.yaml

Поскольку в манифесте Deployments я указал две реплики приложения для обеспечения его отказоустойчивости, мне потребуется балансировщик нагрузки.

Пишу код Terraform для создания балансировщика нагрузки. Создается группа балансировщика нагрузки, которая будет использоваться для 
балансировки нагрузки между экземплярами. Создается балансировщик с именем grafana, определяется слушатель на порту 3000, который 
перенаправляет трафик на порт 30050 целевого узла, настраивается проверка работоспособности (healthcheck) на порту 30050. Также 
создается балансировщик с именем web-app, определяется слушатель на порту 80, который перенаправляет трафик на порт 30051 целевого узла, 
настраивается проверка работоспособности (healthcheck) на порту 30051.

Ссылка на код Terraform балансировщика нагрузки: https://github.com/slava1005/savilov-devops/blob/main/terraform/load-balancer.tf

После применения балансировщика нагрузки к облачной инфраструктуре Outputs выглядит следующим образом:

![img1_51](https://github.com/user-attachments/assets/1134b524-ff25-4f2a-8fae-34ee1370f775)

Проверю работу балансировщика нагрузки. Тестовое приложение будет открываться по порту 80, а Grafana будет открываться по порту 3000:

Тестовое приложение:

![img1_52](https://github.com/user-attachments/assets/f78da900-c363-46a8-8bda-f12a789d5568)

Grafana:

![img1_53](https://github.com/user-attachments/assets/e84df1bc-a897-4fe3-92fc-c73f97d644cc)

Также видно, что в Grafana отобразился созданный Namespace и Deployment с подами.

Развёртывание системы мониторинга и тестового приложения завершено.

### Установка и настройка CI/CD

Для организации процессов CI/CD буду использовать GitLab.

Создаю в GitLab новый пустой проект с именем diplom-test-site.

![img1_54](https://github.com/user-attachments/assets/c05e4433-34cc-45a2-8e4b-5fd03032cf87)

Отправлю созданную ранее статичную страницу и Dockerfile из старого репозитория GitHub в новый проект на GitLab:

![img1_56](https://github.com/user-attachments/assets/1cd1ec23-645f-4f7f-b27e-dfa51ce911e5)

Для автоматизации процесса CI/CD мне нужен GitLab Runner, который будет выполнять задачи, указанные в файле .gitlab-ci.yml.

На странице настроек проекта в разделе подключения GitLab Runner создаю Runner. Указанные на странице данные понадобятся 
для регистрации и аутентификации Runner'а в проекте.

![img1_57](https://github.com/user-attachments/assets/3df78238-5747-4fbf-88f4-7a7c600a2e04)

Выполню подготовку Kubernetes кластера к установке GitLab Runner'а. Создам отдельный Namespace, в котором будет располагаться 
GitLab Runner и создам Kubernetes secret, который будет использоваться для регистрации установленного в дальнейшем GitLab Runner:
```
kubectl create ns gitlab-runner
```
```
kubectl --namespace=gitlab-runner create secret generic runner-secret --from-literal=runner-registration-token="<token>" --from-literal=runner-token=""
```
![img1_58](https://github.com/user-attachments/assets/8ad0a22f-0e7b-4589-a537-f5bfcc27d319)

Также понадобится подготовить файл значений values.yaml, для того, чтобы указать в нем количество Runners, время проверки наличия новых задач, 
настройка логирования, набор правил для доступа к ресурсам Kubernetes, ограничения на ресурсы процессора и памяти.

Файл значений values.yaml, который будет использоваться при установке GitLab Runner доступен по ссылке: https://github.com/slava1005/savilov-devops/blob/main/helm-runner/values.yaml

Приступаю к установке GitLab Runner. Устанавливать буду используя Helm:
```
helm repo add gitlab https://charts.gitlab.io
```
```
helm install gitlab-runner gitlab/gitlab-runner -n gitlab-runner -f helm-runner/values.yaml
```
![img1_59_1](https://github.com/user-attachments/assets/2b4fe2a0-bdcf-4208-b17a-4782dc907f8a)

![img1_59_2](https://github.com/user-attachments/assets/bfe783fa-dc4d-4bab-be84-f60d549e85f0)

Проверю результат установки:

![img1_60_1](https://github.com/user-attachments/assets/46fa3648-be4d-4b45-a1a0-5cfbed019ba0)

GitLab Runner установлен и запущен. Также можно через web-интерфейс проверить, подключился ли GitLab Runner к GitLab репозиторию:

![img1_61](https://github.com/user-attachments/assets/f23af1d7-493f-4cf4-818f-67f61f1190c1)

Подключение GitLab Runner к репозиторию GitLab завершено.

Для выполнения GitLab CI/CD Pipeline мне понадобится в настройках созданного проекта в разделе Variables указать переменные:

![img1_62](https://github.com/user-attachments/assets/722fefdb-b42c-4200-b286-ca78fd77db06)














