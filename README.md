# Домашнее задание "Микросервисы: принципы" - `Фомичев Анатолий`

## Ссылка на Д3 - https://github.com/netology-code/micros-homeworks/blob/main/11-microservices-02-principles.md

## Ссылка на репозиторий - https://github.com/SLzDevOps/netology-microservice-2/tree/main/api-gateway-demo



### Задача 1
```
Выбор решения: Обеспечение разработки (CI/CD)
Предлагаемое решение: GitLab (Self-managed или SaaS)
GitLab — это единая платформа, которая закрывает все требования "из коробки": Git-репозитории, CI/CD, реестр контейнеров, управление секретами, шаблоны и распределённые раннеры.
```
| Требование | Реализация в GitLab |
| :--- | :--- |
Облачная система	GitLab.com (SaaS) или Self-managed в облаке (Yandex Cloud, AWS, GCP)
Система контроля версий Git |	Встроенный Git-репозиторий
Репозиторий на каждый сервис	Отдельный проект или группа проектов
Запуск сборки по событию из VCS	Триггеры на push, merge request, tags
Запуск сборки по кнопке с параметрами	Pipeline schedules +手动 запуск с переменными
Привязка настроек к каждой сборке	Переменные окружения на уровне pipeline, job, группы
Шаблоны для разных конфигураций	include (local, remote), templates, extends
Безопасное хранение секретов	CI/CD Variables (замаскированные, protected) + интеграция с HashiCorp Vault
Несколько конфигураций из одного репозитория	Несколько .gitlab-ci.yml или parent-child pipelines
Кастомные шаги при сборке	Любые shell-команды, скрипты, before_script, after_script
Собственные Docker-образы для сборки	image: myregistry.com/custom-builder:latest
Развернуть агентов сборки на своих серверах	GitLab Runners (Linux, Windows, k8s) с тегами
Параллельный запуск сборок	parallel: в jobs, несколько runner'ов
Параллельный запуск тестов	Разбивка тестов на джобы, needs:, matrix
Архитектура взаимодействия

```
