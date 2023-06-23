# Ansible Playbook for setting server

## Для чего данный плейбук

Данный Playbook устанавливает Docker, docker-compose, net-tools, gitlab-runner. Дает возможность взаимодействовать ${USER} с данными сервисами без прав sudo.
После чего поднимает 3 контейнера, описанных в **templates/docker-compose-struct.yml.j2**

1. container_name: **webserver** | image: **nginx**
2. container_name: **db** | image: **postgres**
3. container_name: **redis** | image: **redis**

## inventory

В файле prod.yml описываем хосты, на которых необходимо выполнять playbook. Меняем значения **ansible_host**, **ansible_user**

```yml
---
srv1:
  hosts:
    srv1:
      ansible_host: ip
      ansible_user: user
      ansible_port: 22
```

## group_vars

Для всех серверов, на которых необходимо выполнить Playbook, задаем необходимые переменные - создаем каталог с названием **(пр. srv2325, srv23188)**, в нем меняем значения **username** и **ansible_sudo_pass** на соответствующие.  

```yml
---
username: "username"
ansible_sudo_pass: "password"
```

## tags

Добавлены 3 tags для запуска plays по отдельности:

1. **docker** - устанавливает docker, docker-compose, net-tools и добавляет пользователя "{{ username }}" в группу docker
2. **gitlab-runner** - устанавливает gitlab-runner, добавляет "{{ username }}" в группу gitlab-runner, gitlab-runner в группу "{{ username }}" и пользователя "{{ username }}" в группу gitlab-runner
3. **struct-up** - создает директории /home/\$USER/docker/nginx/sites-enable, копирует template nginx.conf.j2 по пути /home/\$USER/docker/nginx/nginx.conf, копирует template docker-compose-struct.yml.j2 по адресу /home/\$USER/docker/docker-compose.yml и запускает docker-compose.yml. Данная таска будет корректно выполнена, только при условии установки docker, docker-compose и выдачи необходимых permissions.

## Запуск playbook

Если сервер новый, нам обходимо добавить его в **known_hosts** и прокинуть публичный ключ с исполняемой машины.

Прокидываем публичный ключ:
```sh
ssh-copy-id user@ip
```
Добавляем сервер в known_hosts (делаем попытку подключения к серверу, выбираем "yes"):
```sh
ssh user@ip
```
Из корневой директории выполняем команду для запуска playbook на всех описанных хостах:

```sh
ansible-playbook -i inventory/prod.yml site.yml --diff
```
Если необходимо выполнить только один из plays, используйте флаг --tags [tag] при запуске
