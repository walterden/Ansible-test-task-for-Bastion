# Задание

Задание: Напиши ansible роль, устанавливающую пароль заданного пользователя (пусть будет с именем test) и флаг необходимости смены пароля при первом входе в систему.

# Ответ:

## Создание Ansible role

Необходимо инициализировать ansible role при помощи команды 
`ansible-galaxy role init test_task`, таким образом ansible создаст структуру каталогов в текущем рабочем каталоге.

test_task/
 README.md

 defaults/
 main.yml

 files/

 handlers/
 main.yml
 
 meta/
 main.yml

 tasks/
 main.yml
 
 templates/
 
 tests/
 inventory
 test.yml
 
 vars/
 main.yml

В файле `./teast_task/tasks/main.yml` располагаются необходимые задания, установка пароля заданного пользователя и флаг необходимости смены пароля при первом входе в систему.

``` yml
---
# tasks file for test_task

- name: Ensure the user "{{ username }}" exists # имя задания
  ansible.builtin.user:
    name: "{{ username }}"
    password: "{{ password }}"
    update_password: always

- name: Force password change on first login # имя задания
  ansible.builtin.shell: passwd --expire "{{ username }}"


```

`ansible.builtin.user` - модуль ansible, управляет учетными данными пользователей и пользовательскими атрибутами.
`name` `password` `update_password` - параметры модуля `ansible.builtin.user`

`name` - имя пользователя, которого требуется создать, удалить или изменить.
`password:` - введите хешированный пароль в качестве значения.
`password: always` - обновит пароль если он отличается.
`"{{ username }}"` `"{{ password }}"` - переменные.

Так как в документации модуля ansible `ansible.builtin.user` нету параметра смены пароля при первом входе в систему. Необходимо воспользоваться вторым модулем ansible `ansible.builtin.shell` - принимает имя команды, за которым следует список аргументов, разделенных пробелом.

`passwd --expire "{{ username }}"` - команда выставляет флаг смены пароля при первом входе в систему.


В файле `./teast_task/vars/main.yml` располагаются необходимые переменные для файла `./teast_task/tasks/main.yml`.

`username: test` - имя пользователя из задания.

`password: "{{ 'simple_password' | password_hash('sha512') }}"` - первоначальный пароль и его хэширование. В документации ansible указано использовать хэширование.  
*If provided, set the user’s password to the provided encrypted hash (Linux) or plain text password (macOS). Linux/Unix/POSIX:** Enter the hashed password as the value.

Ansible Role по заданию выполнена.


## Подготовка к запуску роли

Для того чтобы запустить роль, необходимо использовать файл `hosts` и `playbook`.
В каталоге, где происходила инициализация роли, необходимо создать эти файлы.

### Hosts

Файл `hosts` представляет ini-файл конфигурации к подключаемым хостам.

``` ini
### подключение к хосту с использованием переменной client1 через ip и учетную запись с аунтифицацией/авторизацией паролем
client1 ansible_host=192.168.2.84 ansible_user=root ansible_password=simple_password 

### подключение к хосту с использованием переменной =dns_client2= через dns и учетную запись с аунтифицацией/авторизацией паролем
dns_client2 ansible_host=client2 ansible_user=root ansible_password=simple_password

### подключение к хосту с использованием переменной =ssh_client3= через ip и учетную запись аунтифицацией/авторизацией ssh
ssh_client3 ansible_host=192.168.2.84 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa

### подключение к хосту напрямую через ip и учетную запись аунтифицацией/авторизацией ssh
192.168.2.84 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa

### группировка вышеперечисленных хостов
[groupe1]
client1
dns_client2
[groupe2]
ssh_client3
192.168.2.84
[all_groupe:children]
groupe1
groupe2
```

ниже будут представлены какие параметры используются для подключения к хосту.

`client1` `dns_client2` `ssh_client3` - псевдонимы, которые использует ansible для подключения к хосту

`ansible_host` - ip-адрес или dns имя хоста

`ansible_user` - имя пользователя, под которым ansible подключается к хосту

`ansible_password` - пароль, под которым ansible подключается к хосту

`ansible_ssh_private_key_file` - так же можно подключаться с помощью ssh ключа. Необходимо указать путь к ssh ключу.

Допускается не использовать псевдоним и `ansible_host`, а сразу написать  ip-адрес или dns имя целевого хоста.

Вышеперечисленные подключения к хостам можно группировать используя квадратные скобки [], в них указывается имя группы.
Так же группы можно группировать в дочерние группы, используя:
`name_groupe:children` в квадратных скобках.

### Playbook

Переходим к созданию `playbook`.

``` yml
---
- hosts: all
  become: yes
  roles:
    - test_task
```


`hosts` - на каких хостах будет выполняться плейбук. Можно указать псевдоним хоста, ip-адрес, dns имя, группу, или all (выполнит на всех хостах), из файла hosts.

`become` - выполнение плейбука с повышенными привилегиями пользователя. Не всегда необходимо, но в нашем случае необходимо.

`roles` - роль, которую необходимо выполнить.

## Запуск плейбука

Запуск плейбука происходим в директории плейбука при помощи команды:

`ansible-playbook -i hosts playbook.yml`

`-i hosts` - указывает наш hosts файл, по умолчанию ansible берет свой файл из своей конфигурации.

`playbook.yml` - имя нашего плейбука.

Весь проект можно изучить на моем Github


