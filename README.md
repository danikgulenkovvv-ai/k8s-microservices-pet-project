# Kubernetes Microservices Infrastructure & Observability

Практический DevOps-проект по автоматизированному развертыванию и мониторингу микросервисного приложения (**Google Online Boutique**: 11 сервисов на Go, Python, Node.js, C#, Java и база данных Redis) в кластере Kubernetes (k3s).

## Цель проекта
Построить воспроизводимую инфраструктуру, приближенную к продакшен-среде: автоматизировать подготовку сервера, изолировать сервисы внутри кластера, настроить маршрутизацию внешнего трафика, автоскейлинг и сбор метрик.

## Стек технологий
* **ОС и виртуализация:** Ubuntu Linux (VirtualBox)
* **Configuration Management:** Ansible
* **Оркестрация и сеть:** Kubernetes (k3s), Traefik Ingress Controller
* **Пакетный менеджер:** Helm
* **Мониторинг:** Prometheus, Grafana (`kube-prometheus-stack`)
* **CI/CD:** GitHub Actions (`ansible-lint`, `kubeconform`)

## Архитектура и реализованные задачи

1. **Автоматизация развертывания (IaC):**
   Написан Ansible-плейбук (`ansible/setup-k3s.yml`) для автоматической установки одноузлового кластера k3s и настройки конфигурации `kubectl`.
2. **Изоляция и деплой приложения:**
   Микросервисы и Redis развернуты в отдельном пространстве имён `boutique`. Внутреннее взаимодействие между сервисами организовано через объекты `ClusterIP Service` (gRPC).
3. **Маршрутизация трафика (Ingress):**
   Настроены правила `Ingress` для контроллера Traefik, обеспечивающие доступ к веб-интерфейсу магазина (`boutique.local`) и панели мониторинга (`grafana.local`) по доменным именам без ручного проброса портов.
4. **Автомасштабирование (HPA):**
   Сконфигурирован `HorizontalPodAutoscaler` для сервиса `frontend`, автоматически увеличивающий количество реплик при росте утилизации CPU (тестировалось с помощью встроенного генератора нагрузки `loadgenerator`).
5. **Мониторинг и визуализация:**
   С помощью Helm в отдельный неймспейс `monitoring` установлен стек `kube-prometheus-stack`. Настроен сбор метрик с ноды и подов с выводом на дашборды Grafana.
6. **Непрерывная интеграция (CI):**
   В GitHub Actions настроен пайплайн, который при каждом `push` и `pull_request` проверяет синтаксис плейбуков Ansible и валидирует схемы манифестов Kubernetes.

## Структура репозитория
```text
├── ansible/
│   └── setup-k3s.yml           # Плейбук установки и настройки кластера k3s
├── k8s/
│   ├── boutique-manifests.yaml # Манифесты микросервисов и Redis
│   ├── ingress.yaml            # Маршрутизация трафика для магазина
│   ├── grafana-ingress.yaml    # Маршрутизация трафика для Grafana
│   └── hpa.yaml                # Правила горизонтального автоскейлинга
└── .github/workflows/
    └── ci.yml                  # CI-пайплайн валидации кода инфраструктуры
```

## Быстрый запуск

1. **Установка кластера через Ansible:**
   ```bash
   sudo ansible-playbook ansible/setup-k3s.yml
   ```

2. **Деплой приложения и Ingress:**
   ```bash
   kubectl create namespace boutique
   kubectl apply -f k8s/boutique-manifests.yaml -n boutique
   kubectl apply -f k8s/ingress.yaml
   kubectl apply -f k8s/hpa.yaml
   ```

3. **Установка мониторинга:**
   ```bash
   kubectl create namespace monitoring
   helm install prometheus oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack -n monitoring
   kubectl apply -f k8s/grafana-ingress.yaml
   ```

4. **Настройка локального DNS:**
   Добавить в `/etc/hosts` строку `127.0.0.1 boutique.local grafana.local`.
