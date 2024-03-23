# Описание выполненной работы

Результаты =======================================
1) Успешно разобрался с тем как собрать минимальной образ на своей хост машине, образ был успешно запущен в QEMU.
2) Сборка реализована до стадии сборки, на стадии сборки происходит ошибка и образ не собирается (ошибка c
   oe_runmake). Я перепробовал все найденные способы по исправлению данной ошибки, но так и не получил рабочего решения.
   Передача аргументов в контейнер в процессе разработки, данную функцию невозможно будет реализовать пока не будет решена
   проблема с правами пользователя при доступе к смонтированному Docker Volume
4) Реализовал добавление слоя через скрипты bitbake.

Папка с исходниками монитруется к контейнеру и сборка происходит внутри него, но образ создаётся вне docker volume,
из-за чего пока не получается получить собранный образ извне. Это связано с правами доступа при монтировании тома docker,
я пытался решить данную проблему, но на данный момент решение найдено не было.

Файлы, необходимые для решения задачи =============
- get_poky.sh:
  Скрипт для клонирования репозитория дистрибутива Poky и создания локальной ветки версии kirkstone
- add_layer.sh:
  Скрипт для создания слоя Poky, инициализирует среду сборки, создаёт слой и помещает исходный код программы yadro_hello.c
  в папку с рецептом сборки. Вместе с ним идёт файл compile_instr.txt, который содержит инструкции для bitbake, с помощью которых
  программа yadro_hello.c компилируется
- mode_selection.sh:
  Файл, необходимый для выбора режима запуска контейнера.
- entry.sh:
  Файл, объедияющий команды выше (кроме get_poky.sh), который передаётся команде CMD Docker файла.
- Dockerfile:
  - FROM ubuntu:20.04 - сборка на базовом образе ubuntu версии 20.04
  - SHELL \["/bin/bash", "-c"\] - для использования команды source, так как она не работала в стандартном /bin/sh
  - RUN apt update && DEBIAN_FRONTEND=noninteractive ... - обновление репозитория и установка необходимых зависимостей,
    переменная окружения DEBIAN_FRONTEND=noninteractive используется для отключения интерактивного выбора при установе
  - RUN groupadd -gid ... - добавления новой группы с id 1024 и пользователя в этой группе, создание папки пользователя node
  - USER 1024 - переключение на пользователя с id 1024. Новый пользователь нужен так как bitbake не выполняет сборку от
  - пользователя root.
  - WORKDIR /home/node - использование созданной директории пользователя для сборки проекта
  - COPY get_poky.sh ... - для копирования необходимых для сборки скриптов в контейнер
  - CMD source entry.sh - для начала сборки проекта

Инструкция по воспроизведению ========================
1) Выполнить скрипт get_poky.sh с помощью source в папке репозитория, скрипт склонирует репозиторией и создаст локальную ветку с poky версии
kirkstone
2) Создать образ для Docker: docker build -t poky_image .
3) Запустить сборку проекта Poky: "docker run -e mode=build --name poky_cont --rm -v ./poky:/home/node/poky poky_image", режим сборки выбирать
   изменением параметра mode на build - режим сборки, run - режим запуска.
