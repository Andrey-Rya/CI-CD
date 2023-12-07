# Процессы CI/CD

## Урок 3. Continuous delivery и continuous deployment (непрерывная доставка и развертывание)

### Задача

1. Добавить 2 окружения "preprod" и "production"
2. Добавить отдельные deploy job для каждой среды
3. Добавить переменную "$MyLogin" внутри .gitlab-ci.yml, которая будет меняться в зависимости от среды.
4. Добавить переменную "$MyPassword" не используя .gitlab-ci.yml, которая так же будет меняться в зависимости от среды.
5. Добавить скрипт в .gitlab-ci.yml, который `найдёт` все запущенные pipeline по названии ветки(ref) и `остановит` их.

### Решение

### для п. 1-4

Добавляем окружение `preprod` и `production`. В них используем переменную `$MyLogin` (в настройках она будет иметь разные значения).

```yaml
deploy to preprod:
  stage: deploy
  variables:
    TARGET_ENV: preprod
    MyLogin: "Preprod ANDREY"
  script:
    - echo "Do your deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
    - echo "MyLogin"
    - echo $MyLogin
    - echo "MyPassword"
    - echo $MyPassword
  only:
    - main
  environment:
    name: preprod
```

```yaml
deploy to production:
  stage: deploy
  variables:
    TARGET_ENV: production
    MyLogin: "Production ANDREY"
  script:
    - echo "Do your deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
    - echo "MyLogin"
    - echo $MyLogin
    - echo "MyPassword"
    - echo $MyPassword
  only:
    - main
  environment:
    name: production
```

Настройки переменных. Здесь мы добавляем переменную `$MyPassword` через интерфейс. Она тоже будет разной для различных сред исполнения. Также имеет смысл сделать ее Masked (но сейчас так делать не станем, чтобы увидеть корректный вывод переменных). Плюс здесь же добавляем `RUNNER_TOKEN` — он понадобиться для скрипта удаления из пункта 5.

![variables page](img/VirtualBox_cibox_07_12_2023_15_22_42.jpg "variables page")

Pipeline пройден
![pipeline passed](img/VirtualBox_cibox_07_12_2023_15_29_27.jpg "pipeline passed")

Deploy to preprod. На скриншоте — уникальные $MyLogin и $MyPassword для preprod
![deploy to preprod](img/VirtualBox_cibox_07_12_2023_15_31_48.jpg "deploy to preprod")

Deploy to production. Для production — свои $MyLogin и $MyPassword
![deploy to production](img/VirtualBox_cibox_07_12_2023_15_39_31.jpg "deploy to production")

### Пункт 5

```yaml
# ------- Cancel -------
cancel:
  stage: stop previous jobs
  image: everpeace/curl-jq
  script:
    - |
      if [ "$CI_COMMIT_REF_NAME" == "main" ]
        then
          (
            echo "Cancel old pipelines from the same branch except last"
            OLD_PIPELINES=$( curl -s -H "PRIVATE-TOKEN: $RUNNER_TOKEN" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines?ref=${CI_COMMIT_REF_NAME}&status=running" \
                  | jq '.[] | .id' | tail -n +2 )
                  for pipeline in ${OLD_PIPELINES}; \
                      do echo "Killing ${pipeline}" && \
                        curl -s --request POST -H "PRIVATE-TOKEN: ${RUNNER_TOKEN}" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines/${pipeline}/cancel"; done
          ) || echo "Canceling old pipelines (${OLD_PIPELINES}) failed"
      fi
```

Здесь скорее всего нужно разбираться, потому что Pipeline passed, но строки с 34 все-равно вызывают сомнения.

![deploy to production](img/VirtualBox_cibox_07_12_2023_15_44_57.jpg "deploy to production")
