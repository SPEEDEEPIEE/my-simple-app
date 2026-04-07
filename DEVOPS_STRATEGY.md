# Стратегия интеграции Agile и DevOps: CloudStartup MVP

## 1. Контекст проекта
- Цель: ускорить доставку фич, снизить ручной труд, внедрить мониторинг
- Команда: 3 человека (2 dev, 1 ops)
- Стек: PHP, Docker, Jenkins, Prometheus+Grafana

## 2. Процесс разработки
- Спринт: 2 недели
- CI/CD: автоматический для feature-веток, полуавтоматический для main
- DoD: тесты + quality gate + feature flag

## 3. DevOps-практики
- CI (Jenkins)
- CD (автодеплой на staging, ручной на prod)
- Мониторинг (Prometheus+Grafana)
- Алертинг (Telegram)
- Feature Flags

## 4. Метрики успеха
- Deployment Frequency ↑
- Lead Time ↓
- MTTR ↓
- Change Failure Rate ↓

## 5. План следующего спринта
- [ ] Автоматический деплой в prod по тегу release
- [ ] Кэширование в Docker
- [ ] Базовые дашборды в Grafana