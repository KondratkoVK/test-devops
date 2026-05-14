# Test DevOps — Ansible role for Grafana deployment

Данный проект представляет собой Ansible-роль для автоматического развертывания Grafana в Docker.

## Что делает проект

Playbook выполняет следующие действия:

1. Устанавливает Docker и Docker Compose.
2. Запускает службу Docker.
3. Создаёт необходимые директории:
   - `/opt/grafana`
   - `/opt/grafana/config`
   - `/opt/grafana/data`
4. Генерирует конфигурационный файл `grafana.ini` из шаблона.
5. Генерирует файл `docker-compose.yml` из шаблона.
6. Назначает права на каталог данных (`UID 472`).
7. Запускает контейнер Grafana.
8. Проверяет доступность Grafana по HTTP.

## Структура проекта

```text
test-devops/
├── inventory/
│   └── hosts.yml
├── group_vars/
│   └── grafana.yml
├── roles/
│   └── grafana/
│       ├── defaults/
│       │   └── main.yml
│       ├── tasks/
│       │   ├── main.yml
│       │   ├── prepare.yml
│       │   ├── configure.yml
│       │   └── verify.yml
│       └── templates/
│           ├── grafana.ini.j2
│           └── docker-compose.yml.j2
└── playbook.yml
```
## Требования

На целевой машине должны быть установлены:

- Linux (Ubuntu 22.04/24.04)
- Python 3
- Ansible

## Переменные

Основные параметры находятся в файле `group_vars/grafana.yml`:

```yaml
http_port: 3000
domain: localhost
root_url: http://localhost:3000
grafana_dir: /opt/grafana
```

## Запуск playbook

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --ask-become-pass
```

## Запуск отдельных этапов

### Только подготовка

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --tags prepare --ask-become-pass
```

### Только конфигурирование

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --tags configure --ask-become-pass
```

### Только проверка

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --tags verify --ask-become-pass
```

## Проверка результата

После выполнения Grafana будет доступна по адресу:

- `http://localhost:3000` — если открывать на самой машине
- `http://<server_ip>:3000` — если открывать с другого компьютера

Логин и пароль по умолчанию:

- `admin`
- `admin`

## Проверка с нуля

```bash
sudo docker rm -f grafana
sudo rm -rf /opt/grafana

ansible-playbook -i inventory/hosts.yml playbook.yml --ask-become-pass
```

## Используемые технологии

- Ansible
- Docker
- Docker Compose
- Grafana

## Примечание

Во время тестирования была выявлена проблема с правами доступа к каталогу `/opt/grafana/data`. Для её решения в роль добавлена автоматическая установка владельца `472:472`, под которым работает Grafana внутри контейнера.