### Подготовка к выполнению.


Я использую ранее созданный репозиторий для выполнения домашних работ по ansibele, поэтому ничего не надо было делать, кроме как скопировать из репозитория playbook.

Все окружение, vector и clickhouse, у меня будут развернуто в docker-e. 

# Задача 1 

"Подготовьте свой inventory-файл prod.yml."


Измененный prod.yml файл:

```
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: clickhouse
      ansible_connection: docker
vector:
  hosts:
    vector-01:
      ansible_host: vector
      ansible_connection: docker
```

# Задача 2

"Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает vector. Конфигурация vector должна деплоиться через template файл jinja2."

Измененный site.yml файл

```
- name: Install and configure Vector
  hosts: vector
  vars:
    vector_version: "0.34.0"
    vector_install_dir: "/opt/vector"
    vector_config_dir: "/etc/vector"
  
  handlers:
    - name: Restart Vector
      become: true
      ansible.builtin.systemd:
        name: vector
        state: restarted

  tasks:
    - name: Create installation directory
      ansible.builtin.file:
        path: "{{ vector_install_dir }}"
        state: directory
        mode: '0755'

    - name: Download Vector
      ansible.builtin.get_url:
        url: "https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-x86_64-unknown-linux-gnu.tar.gz"
        dest: "/tmp/vector-{{ vector_version }}.tar.gz"
        checksum: "sha256:9a5a6d0b9a9eb7a9f3f3a3e3e3e3e3e3e3e3e3e3e3e3e3e3e3e3e3e3e3e3e3e3"

    - name: Extract Vector
      ansible.builtin.unarchive:
        src: "/tmp/vector-{{ vector_version }}.tar.gz"
        dest: "{{ vector_install_dir }}"
        remote_src: yes
        extra_opts: ["--strip-components=1"]

    - name: Create symlink to binary
      ansible.builtin.file:
        src: "{{ vector_install_dir }}/bin/vector"
        dest: "/usr/local/bin/vector"
        state: link
        force: yes

    - name: Create config directory
      ansible.builtin.file:
        path: "{{ vector_config_dir }}"
        state: directory
        mode: '0755'

    - name: Deploy Vector config
      ansible.builtin.template:
        src: templates/vector.yaml.j2
        dest: "{{ vector_config_dir }}/vector.yaml"
        mode: '0644'
      notify: Restart Vector

    - name: Create systemd service
      ansible.builtin.template:
        src: templates/vector.service.j2
        dest: /etc/systemd/system/vector.service
        mode: '0644'
      notify: Restart Vector

    - name: Ensure Vector is enabled and started
      become: true
      ansible.builtin.systemd:
        name: vector
        state: started
        enabled: yes
        daemon_reload: yes
```

Создал файл в директории templates, vector.yaml.2

```
sources:
  demo_logs:
    type: demo_logs
    format: apache_common
    interval: 1

sinks:
  clickhouse:
    type: clickhouse
    inputs:
      - demo_logs
    database: logs
    table: access_logs
    endpoint: http://clickhouse:8123
    skip_unknown_fields: true
    encoding:
      codec: json
```

Создал файл vector.service.j2, systemd unit-файл, который превращает Vector в управляемый сервис с автоматическим запуском, мониторингом и перезагрузкой при сбоях

```
[Unit]
Description=Vector
Documentation=https://vector.dev
After=network-online.target
Requires=network-online.target

[Service]
User=root
Group=root
ExecStart=/usr/local/bin/vector --config {{ vector_config_dir }}/vector.yaml
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
AmbientCapabilities=CAP_NET_BIND_SERVICE
EnvironmentFile=-/etc/default/vector
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

