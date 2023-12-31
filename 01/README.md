# Процессы CI/CD

## Урок 1. Основы CI/CD. Знакомство с gitlab

### Задача

1. Зарегистрироваться на gitlab.com
2. Создать pipeline и runner
3. Попробовать сохранить артефакт одной из стадий + исключить из папки с артефактами любой файл
4. Попробовать сделать любую gitlab pages

### Решение

1. Сразу проблема с регистрацией из России: поставил gitlab на локальную виртуальную машину
2. Файл .gitlab-ci.yml

```yaml
image: alpine:latest

pages:
  stage: deploy
  script:
    - echo 'First page 01'
    - mkdir .public
    - cp -r * .public
    - mv .public public
  artifacts:
    paths:
      - public
    exclude:
      - public/file_to_exclude.md
  only:
    - master
```

Gitlab Runner также развернут на той же виртуальной машине, запущен для проекта. Статус: passed.

3. Исключение файла из папки с артефактами:

```yaml
---
artifacts:
  paths:
    - public
  exclude:
    - public/file_to_exclude.md
```

4. Снова вышел gitlab pages… Epic fail! После некоторых танцев с бубном - результат все-таки был достигнут. И файлы index.html и style.css были обработаны, и file_to_exclude.md — отброшен.

### Скриншоты

![first gitlab page. project](img/VirtualBox_xbox_03_12_2023_12_49_58.jpg "first gitlab page. project")

![first gitlab page. yaml](img/VirtualBox_xbox_03_12_2023_12_50_56.jpg "first gitlab page. yaml")

![first gitlab page. pipeline passed](img/VirtualBox_xbox_03_12_2023_12_51_37.jpg "first gitlab page. pipeline passed")

![first gitlab page. page folder](img/VirtualBox_xbox_03_12_2023_13_19_40.jpg "first gitlab page. page folder")
