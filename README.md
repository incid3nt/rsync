# Домашнее задание к занятию "Резервное копирование" - Еноктаев Олег


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

Задание 1
- Составьте команду rsync, которая позволяет создавать зеркальную копию домашней директории пользователя в директорию /tmp/backup
- Необходимо исключить из синхронизации все директории, начинающиеся с точки (скрытые)
- Необходимо сделать так, чтобы rsync подсчитывал хэш-суммы для всех файлов, даже если их время модификации и размер идентичны в источнике и приемнике.
- На проверку направить скриншот с командой и результатом ее выполнения

---

Задание 2
- Написать скрипт и настроить задачу на регулярное резервное копирование домашней директории пользователя с помощью rsync и cron.
- Резервная копия должна быть полностью зеркальной
- Резервная копия должна создаваться раз в день, в системном логе должна появляться запись об успешном или неуспешном выполнении операции
- Резервная копия размещается локально, в директории /tmp/backup
- На проверку направить файл crontab и скриншот с результатом работы утилиты.

---

Задания со звёздочкой*
Эти задания дополнительные. Их можно не выполнять. На зачёт это не повлияет. Вы можете их выполнить, если хотите глубже разобраться в материале.

Задание 3*
- Настройте ограничение на используемую пропускную способность rsync до 1 Мбит/c
- Проверьте настройку, синхронизируя большой файл между двумя серверами
- На проверку направьте команду и результат ее выполнения в виде скриншота
---

Задание 4*
- Напишите скрипт, который будет производить инкрементное резервное копирование домашней директории пользователя с помощью rsync на другой сервер
- Скрипт должен удалять старые резервные копии (сохранять только последние 5 штук)
- Напишите скрипт управления резервными копиями, в нем можно выбрать резервную копию и данные восстановятся к состоянию на момент создания данной резервной копии.
- На проверку направьте скрипт и скриншоты, демонстрирующие его работу в различных сценариях.

1. Задание 1 ответ:
Бэкап файлов домашней директории в папку /tmp/backup
```
rsync -a --progress . /tmp/backup
```
посмотрим что получилось, как видим все как нужно:
```
oleg@rsync:~$ ls -al /tmp/backup/netology/ && ls -al netology/
итого 8
drwxr-xr-x 2 oleg oleg 4096 янв 25 23:23 .
drwx------ 3 oleg oleg 4096 янв 25 23:22 ..
-rw-r--r-- 1 oleg oleg    0 янв 25 23:23 test
итого 8
drwxr-xr-x 2 oleg oleg 4096 янв 25 23:23 .
drwx------ 3 oleg oleg 4096 янв 25 23:22 ..
-rw-r--r-- 1 oleg oleg    0 янв 25 23:23 test
```

Исключаем из синхронизации все директории, начинающиеся с точки (скрытые)
```
rsync -av --exclude='.*' . /tmp/backup
```
Необходимо сделать так, чтобы rsync подсчитывал хэш-суммы для всех файлов, даже если их время модификации и размер идентичны в источнике и приемнике.
```
rsync -avc --exclude='.*' . /tmp/backup
```
-c — включение режима проверки контрольных сумм.

2. Скрипт :

```
#!/bin/bash

# Определяем переменные для источника и цели
SOURCE_DIR="/home/oleg"
DESTINATION_DIR="/tmp/backup"

# Дата и время для именования резервной копии
TODAY=$(date +"%Y-%m-%d_%H:%M")

# Создаем директорию для текущей резервной копии
mkdir -p "${DESTINATION_DIR}/${TODAY}"

# Выполняем rsync для создания зеркального бэкапа
rsync -av --delete --exclude='.*' "${SOURCE_DIR}" "${DESTINATION_DIR}/${TODAY}"

# Проверяем статус выполнения rsync
if [ $? -eq 0 ]; then
  echo "Бэкап выполнен успешно, время бэкапа $(date)" >> /var/log/backup.log
else
  echo "Бэкап не выпонен $(date)" >> /var/log/backup.log
fi
```
```
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
0 0 * * * /home/oleg/backup_home.sh
```
```
oleg@rsync:~$  ./backup_home.sh
sending incremental file list
oleg/
oleg/backup_home.sh
oleg/netology/
oleg/netology/test

sent 1.114 bytes  received 70 bytes  2.368,00 bytes/sec
total size is 834  speedup is 0,70
oleg@rsync:~$ ls -al /tmp/backup
итого 44
drwx------ 11 oleg oleg 4096 янв 26 00:25 .
drwxrwxrwt  9 root root 4096 янв 26 00:18 ..
drwxr-xr-x  3 oleg oleg 4096 янв 26 00:02 2025-01-26_00:02
drwxr-xr-x  3 root root 4096 янв 26 00:03 2025-01-26_00:03
drwxr-xr-x  3 root root 4096 янв 26 00:05 2025-01-26_00:05
drwxr-xr-x  3 root root 4096 янв 26 00:09 2025-01-26_00:09
drwxr-xr-x  3 root root 4096 янв 26 00:10 2025-01-26_00:10
drwxr-xr-x  3 oleg oleg 4096 янв 26 00:19 2025-01-26_00:19
drwxr-xr-x  3 oleg oleg 4096 янв 26 00:24 2025-01-26_00:24
drwxr-xr-x  3 oleg oleg 4096 янв 26 00:25 2025-01-26_00:25
drwxr-xr-x  2 oleg oleg 4096 янв 25 23:23 netology
oleg@rsync:~$ tail -f /var/log/backup.log
Backup completed successfully at Вс 26 янв 2025 00:02:27 MSK
Backup completed successfully at Вс 26 янв 2025 00:03:36 MSK
Backup completed successfully at Вс 26 янв 2025 00:05:52 MSK
Бэкап выполнен успешно Вс 26 янв 2025 00:09:22 MSK
Бэкап выполнен успешно, время бэкапа Вс 26 янв 2025 00:10:08 MSK
Бэкап выполнен успешно, время бэкапа Вс 26 янв 2025 00:25:22 MSK
^C
oleg@rsync:~$
```























```
oleg@rsync:~$ ls -al /tmp/backup
итого 12
drwx------ 3 oleg oleg 4096 янв 25 23:34 .
drwxrwxrwt 9 root root 4096 янв 25 23:33 ..
drwxr-xr-x 2 oleg oleg 4096 янв 25 23:23 netology
-rw-r--r-- 1 oleg oleg    0 янв 25 23:34 oleg
oleg@rsync:~$ ls -al
итого 12
drwx------ 3 oleg oleg 4096 янв 25 23:34 .
drwxr-xr-x 4 root root 4096 янв 25 23:21 ..
drwxr-xr-x 2 oleg oleg 4096 янв 25 23:23 netology
-rw-r--r-- 1 oleg oleg    0 янв 25 23:34 oleg
oleg@rsync:~$ rm -rf oleg
oleg@rsync:~$ ls
netology
oleg@rsync:~$ rsync -avc --exclude='.*' . /tmp/backup
sending incremental file list
./

sent 137 bytes  received 20 bytes  314,00 bytes/sec
total size is 0  speedup is 0,00
oleg@rsync:~$ ls -al /tmp/backup
итого 12
drwx------ 3 oleg oleg 4096 янв 25 23:39 .
drwxrwxrwt 9 root root 4096 янв 25 23:33 ..
drwxr-xr-x 2 oleg oleg 4096 янв 25 23:23 netology
-rw-r--r-- 1 oleg oleg    0 янв 25 23:34 oleg
oleg@rsync:~$
```
```
oleg@rsync:~$ rsync -avc --exclude='.*' --delete . /tmp/backup
sending incremental file list
deleting oleg

sent 130 bytes  received 21 bytes  302,00 bytes/sec
total size is 0  speedup is 0,00
oleg@rsync:~$ ls -al /tmp/backup
итого 12
drwx------ 3 oleg oleg 4096 янв 25 23:39 .
drwxrwxrwt 9 root root 4096 янв 25 23:33 ..
drwxr-xr-x 2 oleg oleg 4096 янв 25 23:23 netology
oleg@rsync:~$
```
