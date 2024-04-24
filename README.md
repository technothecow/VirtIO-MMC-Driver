# Virtio MMC Driver

В данном репозитории лежат все файлы, что были подвергнуты изменениям.

Информация об изменениях была взята со следующих страниц: [QEMU](https://gitlab.com/technothecow/qemu/-/compare/master...virtio-mmc?from_project_id=53824584&straight=false), [Linux](https://github.com/torvalds/linux/compare/master...technothecow:linux:mmc-driver)

# Как собрать?

Потребуется система под управлением ОС Linux. Тестировалось на Ubuntu 22.04

1) Склонировать форки [Linux](https://github.com/technothecow/linux) и [QEMU](https://gitlab.com/technothecow/qemu).

2) Собрать линукс:
 - Устранить [зависимости](https://www.kernel.org/doc/html/v6.8-rc5/process/changes.html)
 - Перейти на ветку `mmc-driver`
 - В склонированной директории прописываем `make defconfig` для генерации конфига
 - В файле `.config` находим строчку с `# CONFIG_MMC is not set` и меняем ее на `CONFIG_MMC=y`
 - Прописываем `make oldconfig` и на вопросы отвечаем `y` на `MMC_BLOCK`,  ничего на `MMC_BLOCK_MINORS`, `m` на `MMC_VIRTIO` и `n` на все остальное
 - Запускаем сборку с помощью `make -jN`, где `N` - число ядер, которые не жалко отдать под сборку

3) Сделать `initrd`
 - Загрузить `docker`
 - Перейти в папку `Docker`
 - Запустить команду из `magic.sh`: в результате у нас в системе появится контейнер с тегом `virtio-kernel-deploy`
 - Перейти в папку `Kernel`
 - Запустить команду `docker run --platform=linux/amd64 -v {путь до ядра}:/build/ -v {путь до ядра}:/usr/src/linux -v {путь до текущей папки}:/boot --rm -it virtio-kernel-deploy`
 - Нас интересуют получившиеся файлы `initrd` и `vmlinuz`

4) Собрать QEMU:
 - Создать в корневой директории репозитория папку `build`
 - Запустить следующую команду: `{путь до QEMU}/configure --target-list=x86_64-softmmu`
 - Запускаем сборку с помощью `make -jN`, где `N` - число ядер, которые не жалко отдать под сборку

5) Подготовить образ:
 - Открыть корневую директорию репозитория
 - Прописать следующую команду: `dd if=/dev/zero of=./mmc.img bs=1M count=1`

# Как запустить?

## Если все собрали

Заходим в папку `build` и прописываем следующую команду: `./qemu-system-x86_64 -M q35 -m 256 -kernel ../Kernel/vmlinuz -initrd ../Kernel/initrd -append "console=ttyS0,115200 nokaslr" -serial stdio -display none -drive file=../mmc.img,if=none,id=my_mmc -device virtio-mmc-pci,id=mmc0`

## Если скачали папку с релизом

Заходим в папку с релизом и пишем следующую команду: `./qemu-system-x86_64 -M q35 -m 256 -kernel ./vmlinuz -initrd ./initrd -append "console=ttyS0,115200 nokaslr" -serial stdio -display none -drive file=./mmc.img,if=none,id=my_mmc -device virtio-mmc-pci,id=mmc0`