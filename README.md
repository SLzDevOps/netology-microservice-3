# Домашнее задание "Микросервисы: подходы" - `Фомичев Анатолий`

## Ссылка на Д3 - https://github.com/netology-code/micros-homeworks/blob/main/11-microservices-03-approaches.md


### Задача 1
```
Выбор решения: Обеспечение разработки (CI/CD)
Предлагаемое решение: GitLab (Self-managed или SaaS)
GitLab — это единая платформа, которая закрывает все требования "из коробки": Git-репозитории,
CI/CD, реестр контейнеров, управление секретами, шаблоны и распределённые раннеры.
```

| Требование | Реализация в GitLab |
| :--- | :--- |
Облачная система | GitLab.com (SaaS) или Self-managed в облаке (Yandex Cloud, AWS, GCP)
Система контроля версий Git |	Встроенный Git-репозиторий
Репозиторий на каждый сервис | Отдельный проект или группа проектов
Запуск сборки по событию из VCS | Триггеры на push, merge request, tags
Запуск сборки по кнопке с параметрами | Pipeline schedules + ручной запуск с переменными
Привязка настроек к каждой сборке | Переменные окружения на уровне pipeline, job, группы
Шаблоны для разных конфигураций | include (local, remote), templates, extends
Безопасное хранение секретов | CI/CD Variables (замаскированные, protected) + интеграция с HashiCorp Vault
Несколько конфигураций из одного репозитория | Несколько .gitlab-ci.yml или parent-child pipelines
Кастомные шаги при сборке | Любые shell-команды, скрипты, before_script, after_script
Собственные Docker-образы для сборки | image: myregistry.com/custom-builder:latest
Развернуть агентов сборки на своих серверах  | GitLab Runners (Linux, Windows, k8s) с тегами
Параллельный запуск сборок | parallel: в jobs, несколько runner'ов
Параллельный запуск тестов | Разбивка тестов на джобы, needs:, matrix

```
Архитектура взаимодействия
    Dev[Разработчик] -->|git push| GitLab[GitLab Server]
    GitLab -->|триггер| Runner[GitLab Runner on k8s/VM]
    Runner -->|build| Docker[Docker image]
    Runner -->|push| Registry[Container Registry]
    Runner -->|deploy| K8s[Kubernetes Cluster]
    Registry -->|pull| K8s
    Dev -->|manual run| UI[GitLab UI]

Обоснование выбора: 
- Единый стек — не нужно интегрировать Jenkins, GitHub, Nexus, Vault отдельно.
- Родная поддержка микросервисов — монорепозиторий или мульти-репозиторий, parent-child pipelines.
- Безопасность — CI/CD variables + Vault integration + защита веток.
- Масштабирование — авто-скейлинг runner'ов в Kubernetes (GitLab Operator).
- Удобство для команды — привычный интерфейс, code review, CI/CD embedded.
```

### Задача 2
```
Логи
Предлагаемое решение: Vector + OpenSearch (Elasticsearch) + OpenSearch Dashboards (Kibana)
```
| Компонент | Роль |
| :--- | :--- |
Vector | Сбор, обогащение, буферизация и доставка логов со всех хостов
OpenSearch | Центральное хранилище (замена Elasticsearch)
OpenSearch Dashboards | UI для поиска, фильтрации, дашбордов

```
Архитектура взаимодействия
    App1[Сервис 1] -->|stdout| Vector1[Vector Agent]
    App2[Сервис 2] -->|stdout| Vector2[Vector Agent]
    Vector1 -->|buffer| Kafka[Kafka / Redis]
    Vector2 -->|buffer| Kafka
    Kafka -->|consumer| VectorAgg[Vector Aggregator]
    VectorAgg -->|batch| OpenSearch[(OpenSearch)]
    OpenSearch --> Dashboards[OpenSearch Dashboards]
    Dashboards -->|port 5601| Dev[Разработчик]

Соответствие требованиям
```
| Требование | Решение |
| :--- | :--- |
Сбор логов в центральное хранилище | Vector aggregator → OpenSearch
Сбор из stdout | Vector source docker_logs или file с перехватом stdout
Гарантированная доставка | Buffer на диске (Vector) + acknowledgement от OpenSearch
Поиск и фильтрация | OpenSearch Dashboards (KQL, Lucene)
UI для доступа разработчикам | Роли в OpenSearch Security Plugin
Ссылка на сохранённый поиск | Сохранение поисков в Dashboards (URL share)

```
Обоснование выбора:
- Vector вместо Logstash — написан на Rust, потребляет меньше ресурсов, поддерживает буферизацию на диск, гарантирует доставку.
- OpenSearch вместо Elasticsearch — форк Elasticsearch с открытой лицензией, совместим с Kibana-плагинами.
- Минимальные требования к приложениям — только писать логи в stdout (Docker-контейнеры).
- Гарантированная доставка — благодаря дисковому буферу Vector и подтверждениям записи.
```

### Задача 3
```
Мониторинг
Предлагаемое решение: Prometheus + Grafana + Node Exporter + cAdvisor + Metrics exporters
```

| Компонент | Роль |
| :--- | :--- |
Prometheus | Сбор метрик, хранение, алертинг
Grafana | Визуализация, дашборды
Node Exporter | Метрики хостов (CPU, RAM, HDD, Network)
cAdvisor | Метрики контейнеров (для каждого сервиса)
Blackbox Exporter | Проверка доступности сервисов
Пользовательские exporters | Метрики приложений (например, /metrics в Python/Go)

```
Архитектура взаимодействия

    Host1[Хост 1] --> NodeExp[Node Exporter]
    Host1 --> cAdvisor[cAdvisor]
    Host1 --> AppExp[App /metrics]
    Host2 --> NodeExp2[Node Exporter]
    Host2 --> cAdvisor2[cAdvisor]
    Prometheus -->|pull| NodeExp
    Prometheus -->|pull| cAdvisor
    Prometheus -->|pull| AppExp
    Prometheus-->|store| TSDB[(TSDB)]
    Grafana -->|query| Prom
    Grafana --> UI[Дашборды]

Соответствие требованиям
```
| Требование | Решение |
| :--- | :--- |
Сбор метрик со всех хостов | Prometheus с конфигом file_sd_config или kubernetes_sd_config
Метрики ресурсов хостов | Node Exporter (CPU, RAM, HDD, Network)
Метрики ресурсов на сервис | cAdvisor (через Docker/k8s) + Node Exporter
Специфичные метрики сервисов | /metrics endpoint (Prometheus client libraries)
UI с запросами и агрегацией | Grafana (Explore)
Настраиваемые панели | Grafana Dashboards (шаблоны, переменные)

```
Обоснование выбора:
- Prometheus — стандарт индустрии для микросервисов (особенно в Kubernetes).
- Grafana даёт гибкие дашборды, алерты, аннотации, легко делиться ссылками.
- Экспортеры покрывают все уровни: хост (Node), контейнеры (cAdvisor), приложения (свои).
- Pull-модель упрощает обнаружение сервисов (Service Discovery) в динамической среде.
```

#### Итоговая таблица
| Задача | Решение | Ключевые компоненты |
| :--- | :--- | :--- |
CI/CD | GitLab (Self-managed / SaaS) | Git, CI/CD pipelines, Runners, Container Registry, Vault
Логи | Vector + OpenSearch + OpenSearch Dashboards | Vector агенты, Kafka-буфер, OpenSearch кластер
Мониторинг | Prometheus + Grafana + exporters | Node Exporter, cAdvisor, Blackbox, кастомные метрики













