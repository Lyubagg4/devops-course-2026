# Server setup

Учебная документация целевого сервера. Пароли и закрытые ключи в документе не хранятся.

## 10. Ansible

**Расположение проекта:** каталог `ansible/` репозитория `devops-course-2026` (ветка `feature/ansible`).

**Узлы:**
- управляющий — macOS (Ansible, коллекции, ключ `~/.ssh/devops_vm`);
- управляемый — VM `devops-vm`, Ubuntu Server 24.04 (UTM, архитектура arm64 — адаптация под Apple Silicon), адрес `192.168.64.3`.

**Inventory:**
- `inventory/bootstrap.ini` — первичный запуск от пользователя `lyuba` по паролю (`--ask-pass --ask-become-pass`);
- `inventory/prod.ini` — последующие запуски от `devops` по ключу `~/.ssh/devops_vm`, неинтерактивно (ansible.cfg подключает prod.ini по умолчанию).

**Роли:** `base` (проверка ОС, пакеты, репозиторий Docker, служба), `ssh` (пользователь devops, ключ, sudo NOPASSWD, hardening SSH, UFW), `nginx` (reverse proxy с проверкой `nginx -t` перед reload), `app` (загрузка образа, контейнер, health-check).

**Приложение:**
- образ: `ghcr.io/lyubagg4/devops-ci-app` (мультиархитектурный: amd64 + arm64);
- тег (полный SHA зелёного коммита из ПР № 10): `0871266dcb056aa724dd3acaef02f131aac1ba50`;
- порт контейнера: опубликован только на `127.0.0.1:5000` (доступ извне — только через Nginx на 80);
- health endpoint: `/health` → `{"status": "ok"}`.

**Команды:**
- первичный запуск: `ansible-playbook -i inventory/bootstrap.ini site.yml --ask-pass --ask-become-pass`
- повторный запуск (идемпотентный, `changed=0`): `ansible-playbook site.yml`
- read-only аудит: `ansible-playbook verify.yml`
