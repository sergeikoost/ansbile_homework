# Подготовка к выполнению.

## Создайте два пустых публичных репозитория в любом своём проекте: vector-role и lighthouse-role.
## Добавьте публичную часть своего ключа к своему профилю на GitHub.



У меня единый репозиторий для всех домашних работ в ansible, поэтому уже все заранее созданно и будет выполняться тут. Скопировал playbook с прошлой работы в директорию для этой работы (homework_4), при помощи команд ansible-galaxy role init сделал 3 роли для выполнения поставленной задачи -  сделать roles для ClickHouse, Vector и LightHouse и написать playbook для использования этих ролей.





### 1. Cоздал файл  requirements.yml в корневом каталоге playbook.



### 2. При помощи ansible-galaxy скачайте себе эту роль.



Скачал роль и поместил её в каталог с домашней работой №4 при помощи команды ansible-galaxy install -r requirements.yml -p /home/ansbile_homework/homework_4/


![ansible_homework4 1](https://github.com/user-attachments/assets/afe9f07a-fb5e-4f2f-86c8-4b0c354bc5f4)




### 3. Создайте новый каталог с ролью при помощи ansible-galaxy role init vector-role. Повторите шаги 3–6 для LightHouse. Помните, что одна роль должна настраивать один продукт.

ansible-galaxy role init vector-role 
ansible-galaxy role init lighthouse-role

После команды ansible-galaxy role init создается каталог с ролью. 
