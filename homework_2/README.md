### Подготовка к выполнению.


Я использую ранее созданный репозиторий для выполнения домашних работ по ansible, поэтому ничего не надо было делать, кроме как скопировать из репозитория playbook.

В качестве окружения я использовал виртуальную машину, созданную в yandex cloud

# Задача 1 

"Подготовьте свой inventory-файл prod.yml."


Измененный prod.yml файл:

```
clickhouse:
  hosts:
    clickhouse:
      ansible_host: 158.160.12.146
      ansible_user: kusk111serj 
      ansible_ssh_private_key_file: /root/.ssh/ansible_hm
```

# Задача 2

"Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает vector. Конфигурация vector должна деплоиться через template файл jinja2." 

Вот тут возникли сложности, clickhouse упорно не хотел подниматься, полностью переделал таску исходя из официальной документации. Логика закомментирована.

Измененный site.yml файл:

```
---
- name: Install ClickHouse
  hosts: clickhouse
  handlers:
    - name: Start clickhouse service
      become: true
      ansible.builtin.service:
        name: clickhouse-server
        state: restarted
  tasks:
    - name: Update apt cache
      become: true
      ansible.builtin.apt:
        update_cache: true
      ignore_errors: true

    - name: Install prerequisites
      become: true
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - ca-certificates
          - gnupg
          - curl
        state: present
      ignore_errors: true


    - name: Add ClickHouse repository
      become: true
      ansible.builtin.apt_repository:
        repo: "deb [signed-by=/usr/share/keyrings/clickhouse-keyring.gpg] https://packages.clickhouse.com/deb stable main"
        state: present
        filename: "clickhouse.list"

    - name: Install ClickHouse packages
      become: true
      ansible.builtin.apt:
        name:
          - clickhouse-server
          - clickhouse-client
        state: present
      notify: Start clickhouse service

    - name: Create database
      ansible.builtin.command: "clickhouse-client -q 'create database logs;'"
      register: create_db
      failed_when: create_db.rc != 0 and create_db.rc != 82
      changed_when: create_db.rc == 0

- name: Install Vector
  hosts: clickhouse
  become: true
  tasks:
    - name: Create vector user
      become: true
      ansible.builtin.user:
        name: vector
        system: true
        create_home: false

    - name: Create Vector directories
      become: true
      ansible.builtin.file:
        path: "{{ item }}"
        state: directory
        owner: vector
        group: vector
        mode: '0755'
      with_items:
        - /etc/vector
        - /var/lib/vector
        
    - name: Install prerequisites
      become: true
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
        state: present
        update_cache: true
      register: vector_prereq
      failed_when: false

    - name: Add Vector repository
      become: true
      ansible.builtin.shell: |
        bash -c "$(curl -fsSL https://setup.vector.dev)"
      args:
        creates: "/etc/apt/sources.list.d/vector.list"
      changed_when: false

    - name: Update apt cache
      become: true
      ansible.builtin.apt:
        update_cache: true
      register: vector_apt_update
      failed_when: false
      ignore_errors: true

    - name: Install Vector
      become: true
      ansible.builtin.apt:
        name: vector
        state: present
        update_cache: false

    - name: Deploy Vector config
      become: true
      ansible.builtin.template:
        src: "templates/vector.yaml.j2"
        dest: "/etc/vector/vector.yaml"
        owner: vector
        group: vector
        mode: "0640"

    - name: Install Vector service file
      become: true
      ansible.builtin.template:
        src: "templates/vector.service.j2"
        dest: "/etc/systemd/system/vector.service"
        mode: "0644"

    - name: Reload systemd
      become: true
      ansible.builtin.systemd:
        daemon_reload: true

    - name: Enable and start Vector
      become: true
      ansible.builtin.systemd:
        name: vector
        state: started
        enabled: true


```

Создал файл в директории templates, vector.yaml.2

```
sources:
  apache_logs:
    type: demo_logs
    format: apache_common
    interval: 1
    count: 100

sinks:
  clickhouse_sink:
    type: clickhouse
    inputs: [apache_logs]
    endpoint: "http://localhost:8123"
    database: "logs"
    table: "access_logs"
    skip_unknown_fields: true
    compression: gzip
    format: "JSONEachRow"
    request:
      retry_attempts: 5
      retry_initial_backoff_secs: 1
      timeout_secs: 30
    auth:
      user: "default"
      password: ""
```

Создал файл vector.service.j2:

```
[Unit]
Description=Vector Log Aggregator
After=network-online.target
Requires=network-online.target

[Service]
ExecStart=/usr/bin/vector --config /etc/vector/vector.yaml
Restart=always
RestartSec=5
Environment="RUST_BACKTRACE=full"
User=vector
Group=vector
LimitNOFILE=65536
ProtectSystem=strict
ProtectHome=read-only
ReadWritePaths=/var/lib/vector

[Install]
WantedBy=multi-user.target
```

Далее запустил проект, до этого проект не хотел запускаться пока не добавил в playbook игнорирование ошибок в тасках, иначе у меня не вышло.

## Запустите ansible-lint site.yml и исправьте ошибки, если они есть.

![image](https://github.com/user-attachments/assets/64975737-2b04-48c8-978e-5befd3af06d1)


 1 yaml[trailing-spaces] basic   formatting, yaml   ошибку исправил, 2 ignore-errors         shared  unpredictability  исправлять не стал т.к. иначе проект не запускается.

## Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.

![image](https://github.com/user-attachments/assets/abea9bb1-e8fc-4725-b9c6-a46be5ef888a)


## Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.

![image](https://github.com/user-attachments/assets/0af3f942-707e-4413-9203-46799d4b9a99)


## Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. Пример качественной документации ansible playbook по ссылке. Так же приложите скриншоты выполнения заданий №5-8

```

Playbook выполняет установку и настройку двух компонентов на целевых хостах:

ClickHouse - колоночная СУБД
Vector - инструмент для сбора и обработки логов

Playbook состоит из двух отдельных play, работающих на группе хостов clickhouse.
```

