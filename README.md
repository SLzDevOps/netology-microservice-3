# Домашнее задание "Микросервисы: принципы" - `Фомичев Анатолий`

## Ссылка на Д3 - https://github.com/netology-code/micros-homeworks/blob/main/11-microservices-02-principles.md

## Ссылка на репозиторий - https://github.com/SLzDevOps/netology-microservice-2/tree/main/api-gateway-demo



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
OpenSearch | Dashboards	UI для поиска, фильтрации, дашбордов












