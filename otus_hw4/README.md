
####Задание 1 Определение алгоритма с наилучшим сжатием
#Создаём 4 пула из двух дисков в режиме RAID 1
zpool create otus1 mirror /dev/sdb /dev/sdc
zpool create otus2 mirror /dev/sdd /dev/sde
zpool create otus3 mirror /dev/sdf /dev/sdg
zpool create otus4 mirror /dev/sdh /dev/sdi

#Добавим разные алгоритмы сжатия в каждую файловую систему:
#Алгоритм lzjb: 
zfs set compression=lzjb otus1
#Алгоритм lz4:  
zfs set compression=lz4 otus2
#Алгоритм gzip: 
zfs set compression=gzip-9 otus3
#Алгоритм zle:  
zfs set compression=zle otus4

#Скачаем один и тот же текстовый файл во все пулы (неоптимально, хорошо бы 1 раз скачать, а не 4): 
for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done

#Проверим, сколько места занимает один и тот же файл в разных пулах и проверим степень сжатия файлов c помощью команд zfs list и zfs get all | grep compressratio | grep -v ref
#Таким образом, у нас получается, что алгоритм gzip-9 самый эффективный по сжатию. 
 
 
 
#### Задание 2 Определение настроек пула
#Скачиваем архив в домашний каталог: 
wget -O archive.tar.gz --no-check-certificate 'https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download' 
#Разархивируем его:
tar -xzvf archive.tar.gz
#Проверим, возможно ли импортировать данный каталог в пул:zpool import -d zpoolexport/
#Сделаем импорт данного пула к нам в ОС:
zpool import -d zpoolexport/ otus

#### Задание 3 Работа со снапшотом, поиск сообщения от преподавателя
#Скачаем файл, указанный в задании:
 wget -O otus_task2.file --no-check-certificate "https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download"
 
# Восстановим файловую систему из снапшота: 
 zfs receive otus/test@today < otus_task2.file
 
# Далее, ищем в каталоге /otus/test файл с именем “secret_message”:
find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message

