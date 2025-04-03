### Подготовка к выполнению.


![image](https://github.com/user-attachments/assets/97d3179b-fd79-4798-9193-bc6435a8a697)


Создал публичный репозиторий:
```
git clone https://github.com/sergeikoost/ansbile_homework.git
```

Запушил playbook ранее скачанный с репозитория с домашним заданием.

# Задача 1

Запустил playbook:

![image](https://github.com/user-attachments/assets/85b05248-3c05-46b0-ac0c-28b179c6040c)


# Задача 2

```
cat examp.yml 
---
  some_fact: 'all default fact'
```

![image](https://github.com/user-attachments/assets/820dd6a4-4426-4195-a8fb-efe270d85156)

# Задача 3

Создаем окружение при помощи docker для дальнейшего выполнения заданий:

```
docker run --rm --name "ubuntu" -d pycontribs/ubuntu:latest sleep 3600
docker run --rm --name "centos7" -d pycontribs/centos:7 sleep 3600
```

Выбор пал на образ pycontribs/ubuntu потому что он идеально подходит для для тестирования, CI/CD и разработки, особенно в экосистеме Python и Ansible.

# Задача 4 

Запускаем playbook на окружении из prod.yml

![image](https://github.com/user-attachments/assets/1d5af7d2-7427-48d2-85bf-50463dc23d27)

```
TASK [Print fact]
ok: [centos7] => {
    "msg": "el" # Значение some_fact для хоста centos 7
}
ok: [ubuntu] => {
    "msg": "deb" # Значение some_fact для хоста ubuntu
}
```

# Задача 5


Добавляем факты:

```
cat /home/ansbile_homework/homework_1/playbook/group_vars/deb/examp.yml 
---
  some_fact: "deb default fact"
cat /home/ansbile_homework/homework_1/playbook/group_vars/el/examp.yml 
---
  some_fact: "el default fact"
```

# Задача 6

Производим запуск playbook, видим что значения корректны:

![image](https://github.com/user-attachments/assets/62eca9c5-b442-44c0-a0f5-4d3c7246bfc6)


# Задача 7

Зашифровал факты в group_vars/deb/examp.yml и group_vars/el/examp.yml

```
ansible-vault encrypt group_vars/deb/examp.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful

ansible-vault encrypt group_vars/el/examp.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
```


# Задача 8

Запускаем playbook с ключом --ask-vault-pass

![image](https://github.com/user-attachments/assets/31a35805-2bf6-4c96-89f2-2741d115cfec)

# Задача 9

 Задача требует выбрать плагин для работы на control node (самом Ansible-сервере), значит подойдут только локальные плагины.

```
ansible-doc -t connection --list | grep local #local         execute on controller  
```

# Задача 10

Добавил в prod.yml новую группу хостов

```
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker
  local:
    hosts:
      localhost:
        ansible_connection: local # Локал хосты
```

# Задача 10

Запускаем playbook с ключом --ask-vault-pass

![ansible_homework1](https://github.com/user-attachments/assets/ecc4eacf-0586-4f0e-91e4-1063c9c9f971)


Все значения верны.
