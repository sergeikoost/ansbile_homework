### Подготовка к выполнению.

1) Подготовьте в Yandex Cloud три хоста: для clickhouse, для vector и для lighthouse.

2) Репозиторий LightHouse находится по ссылке. 

Репозиторий этот уже неактуальный "This repository was archived by the owner on Apr 23, 2024. It is now read-only.", но задачу сделал, хоть и не работает ругаясь на то, что такого репозитория нет :)

```
TASK [Download Lighthouse static] **********************************************************************************************************************************************************
fatal: [lighthouse]: FAILED! => {"changed": false, "dest": "/tmp/lighthouse.zip", "elapsed": 0, "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://github.com/VKCOM/lighthouse/archive/refs/heads/gh-pages.zip"}
```


Обновленный site.yml из предыдущего, код play для lighthouse и его таски:

```
- name: Install Lighthouse
  hosts: lighthouse
  become: true
  tasks:
    - name: Install Nginx and dependencies
      ansible.builtin.apt:
        name:
          - nginx
          - unzip
        state: present
        update_cache: true

    - name: Download Lighthouse static
      ansible.builtin.get_url:
        url: "https://github.com/VKCOM/lighthouse/archive/refs/heads/gh-pages.zip"
        dest: "/tmp/lighthouse.zip"
        mode: "0755"

    - name: Create web directory
      ansible.builtin.file:
        path: /var/www/lighthouse
        state: directory
        mode: "0755"

    - name: Unarchive Lighthouse
      ansible.builtin.unarchive:
        src: "/tmp/lighthouse.zip"
        dest: "/var/www/lighthouse"
        remote_src: true
        extra_opts: ["--strip-components=1"]

    - name: Configure Nginx
      ansible.builtin.template:
        src: "templates/nginx-lh.conf.j2"
        dest: "/etc/nginx/sites-available/lighthouse.conf"
        mode: "0644"
      notify: Restart Nginx

    - name: Enable Nginx site
      ansible.builtin.file:
        src: "/etc/nginx/sites-available/lighthouse.conf"
        dest: "/etc/nginx/sites-enabled/lighthouse.conf"
        state: link
      notify: Restart Nginx

    - name: Disable default site
      ansible.builtin.file:
        path: "/etc/nginx/sites-enabled/default"
        state: absent
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```


Также обновил inventory, потому что стало 3 хоста на которых разворачиваются сервисы:

```
clickhouse:
  hosts:
    clickhouse:
      ansible_host: 158.160.74.86
      ansible_user: kusk111serj 
      ansible_ssh_private_key_file: /root/.ssh/ansible_hm
lighthouse:
  hosts:
    lighthouse:
      ansible_host: 158.160.69.148
      ansible_user: kusk111serj 
      ansible_ssh_private_key_file: /root/.ssh/ansible_hm      
vector:
  hosts:
    vector:
      ansible_host: 158.160.95.231
      ansible_user: kusk111serj 
      ansible_ssh_private_key_file: /root/.ssh/ansible_hm            
```


Создал файл j2 для генерирования конфигурации nginx:

```
server {
    listen 80;
    server_name _;
    root /var/www/lighthouse;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

```

Структура проекта такая:

├── ansible.cfg
├── group_vars/
│   └── clickhouse/
│       └── vars.yml
├── inventory/
│   └── prod.yml
├── templates/
│   ├── vector.service.j2
│   └── vector.yaml.j2
|   └── nginx-lh.j2
└── site.yml

Поскольку задание было необязательным в рабочее состояние не стал приводить. "ВНИМАНИЕ! В связи с изменениями в работе Centos7, Практическое задание не является обязательным."
