# otus_hw1
1) Готовим Vagrantfile согласно методичке
2) Готовим файл centos.json для packer
    меняем параметр iso_url на "iso_url": "https://mirror.yandex.ru/centos/8-stream/isos/x86_64/CentOS-Stream-8-x86_64-20221027-boot.iso" , потому как умучался искать подходящую под методичку, соответственно и контрольную сумму меняем
    добавлен параметр "headless":"true", по рекомендациям из slack
3) Готовим файл ks.cfg
    в файле ks.cfg меняем url --url="https://mirror.yandex.ru/centos/8-stream/BaseOS/x86_64/os/"
    Файл приведенный в методичке - кривой и из коробки неверно работает, например параметр "authconfig" устарел, вместо него надо использовать "authselect", пришлось добавлять
	    %packages
	    @^minimal-environment
	    @standard
	    %end
    так как выбора пакетов не было и анаконда ждала рукопашной команды
4) Добавляем файлы скриптов для настроек после установки ОС scripts/stage-1-kernel-update.sh scripts/stage-2-clean.sh
