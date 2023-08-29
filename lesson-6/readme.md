# Домашка к уроку "Физический уровень PostgreSQL"
После переноса содержимого  /var/lib/postgresql/15/main в /mnt/data и попытке старта postgresql ошибка:
```
Error: /var/lib/postgresql/15/main is not accessible or does not exist
```
отсутствует каталог /var/lib/postgresql/15/main на который ссылается параметр data_directory

В файле /etc/postgresql/15/main/postgresql.conf изменен путь в параметре data_directory с /var/lib/postgresql/15/main на /mnt/data/15/main

После изменения пути кластер запускается, так как месторасположение данных в конфигурационном файле указано теперь верно

задание со звездочкой *
1. на первой виртуалке остановлен кластер sudo -u postgres pg_ctlcluster 15 main stop
2. в консоли яндекс клауда отцеплен диск от первой виртуалки и подключен ко второй

На второй виртуалке

1. примонтирован диск к /mnt/data sudo mount /dev/vdb1 /mnt/data
2. в fstab добавлено /dev/vdb1  /mnt/data ext4 defaults 0 0
3. остановлен кластер sudo -u postgres pg_ctlcluster 15 main stop
4. удалены файлы из /var/lib/postgresql
5. отредактирован /etc/postgresql/15/main/postgresql.conf, указан новый путь /var/lib/postgresql/15/main
6. запущен кластер sudo -u postgres pg_ctlcluster 15 main start - ОК
7. выполнена проверка содержимое ранее созданной таблицы - ОК
