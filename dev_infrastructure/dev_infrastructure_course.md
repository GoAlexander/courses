---
marp: true
author: Alexander Dydychkin
title: Infrastructure for Development Course
theme: default
paginate: true
math: mathjax
# https://github.com/orgs/marp-team/discussions/192
style: |
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }

---

# Инфраструктура для разработки (почти DevOps)
автор: Дыдычкин Александр Алексеевич
email: AlexanderDydychkin@yandex.ru

---

# Вступление

- Почему название такое?
- Это DevOps - курс?
- Курс актуален для аналитиков/разработчиков/инфраструктурщиков? 
- Курс содержит специфику Gitlab?

---

# Проблемы в процессе разработки ПО

---

![Alt text](images/process_cleaned.png)

---

![Alt text](images/process.png)

---

![Alt text](images/agile.webp)

---

### Как разрабатывается ПО?

---

![Alt text](images/dev_process.png)

---

Проблемы на ~~бочку~~ доску!

*(типичная проблема поставки / релиза)*

---

# Решения проблем. CI. DevOps

---

## CI - если больно, то давайте делать ЭТО чаще!

- техническая ретроспектива: раньше был cron + .sh скрипты
---

## DevOps

- команда, а не человек
- сотрудничество команд разработки и эксплуатации
- навыки написания кода
- автоматизация, много автоматизации
- конвееры CI/CD используются активно
- все есть текст / "infrastructure as a code" (git)
- культура взаимодействия
- отлаженный процесс развертывания
- рутина

---

## Ок. Проблемы порешали. А что вообще нужно разработчику?

---

![Alt text](images/infrastructure_bricks.png)
<!-- |                 Theoretical service                |                    Implementation                        |
|:--------------------------------------------------:|:--------------------------------------------------------:|
|            Version Control System (VCS)            |                 Git/Subversion/Mercurial                 |
|                    Code storage                    |               Git/Github/Gitlab/Bitbucket/…              |
|                    Review board                    |              Gerrit/CodeScene/Github/Gitlab              |
|                     CI service                     |   Buildbot/Jenkins/TeamCity/Travis/Gitlab CI/ Github CI  |
|                    Build system                    |          Cmake/ninja/Meson/Github, Gitlab API            |
|                     Test system                    |            Pytest/Google Test/Robot Framework            |
|              Virtualization/containers             |                        QEMU/Docker                       |
|   Soft provisioning/config mngmt                   |                          Ansible                         |
|                  Monitoring system                 |                          Zabbix                          |
| Infrastructure language (infrastructure as a code) |                        Python/bash                       | -->



---
<!-- ------- -->
<!-- PART #2 -->
<!-- ------- -->
---

# Gitlab

> GitLab — веб-инструмент жизненного цикла DevOps с открытым исходным кодом, представляющий систему управления репозиториями кода для Git с собственной вики, системой отслеживания ошибок, CI/CD пайплайном и другими функциями.

Развивается компанией Gitlab Inc. по модели [Open-core](https://en.wikipedia.org/wiki/Open-core_model).

---

![Alt text](images/gitlabui.png)

---

## Почему выбран Gitlab?
- популярность:
  - один из самых популярных инструментов в корпоративной среде;
  - некоторые open source организации тоже используют Gitlab (Gnome, VLC и т. д.);
- all-in-one;
- бесплатен в версии Community Edition (CE) - самое то для команд <=100 человек.

---

## Почему не Github?
Знание построения CI в Gitlab в корпоративной среде перспективнее. Таким образом платформа была выбрана из-за CI подсистемы.

---

## Github Actions VS Gitlab CI
Сильные стороны обеих подсистем.

| Github Actions |  Gitlab CI |
|:--------------:|:----------:|
|   Marketplace  | "includes" |
|                | API богаче |

Почему Jenkins и прочие TeamCity даже не рассматриваются? - "all-in-one", степень интеграции!

---
## Шаг назад. Development process.
- Как вести разработку в команде?

[GitLab flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html)
[Github flow](https://docs.github.com/en/get-started/quickstart/github-flow)

---

## Gitlab workflow demo...
1) Commit & push
2) Branching
3) MR
4) Review

---

## Демо!

---

# Gitlab CI
> Подсистема Gitlab, отвечающая за автоматизация.

- Gitlab server - gitlab-runner
- Pipelines

[Подробнейшая документация](https://docs.gitlab.com/ee/ci/)
Практический минимум:
- https://docs.gitlab.com/ee/ci/yaml/
- https://docs.gitlab.com/ee/ci/variables/predefined_variables.html

---

.gitlab-ci.yml
```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  image: gcc
  script:
    - g++ hello_world.cpp -o hello_world.out
  artifacts:
    paths:
      - hello_world.out

test:
  stage: test
  image: gcc
  script:
    - ./tests.sh

deploy:
  stage: deploy
  image: gcc
  script: echo "Here your deployment script!"
  environment: production
```

---

```cpp
#include <iostream>

int main() {
    std::cout << "Hello World!";
    return 0;
}
```

---

## Демо!

---

## Установка и настройка gitlab-runner (на базе docker)

`gitlab-runner` - клиентское приложение, которое выполняет команды, указанные в `.gitlab-ci.yml` в синтаксическом элементе `script`. Можно установить gitlab-runner непосредственно на машину (читайте дополнительно) или установить с помощью docker'а (см. ниже).

Документация по данному вопросу:
- [Установка gitlab-runner с помощью docker](https://docs.gitlab.com/runner/install/docker.html#option-1-use-local-system-volume-mounts-to-start-the-runner-container)
- [Регистрация gitlab-runner](https://docs.gitlab.com/runner/register/index.html#docker)

---

### Делай раз, делай два...

Создадим gitlab-runner для
1) Перейти по ссылке вида: `https://gitlab.com/USER/YOUR_PROJECT/-/settings/ci_cd`
2) Зайти в "Runners" и нажать "New project runner"
3) Настроить по примеру (для linux-based):

---

![bg right:100% 70%](images/runner_configuration.png)

---

4) Нажать "Create runner"
5) Скопировать токен

---

6) На машине установить gitlab-runner:
```bash
# Установка gitlab-runner с помощью докер-образа
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

7) Начать регистрацию:
```bash
docker run --rm -it -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register

# пример заполнения:
https://gitlab.com
YOUR_TOKEN
LOCAL_RUNNER_NAME
executor: docker (lin) / docker-windows (win)
DEFAULT_IMAGE
```

---

8) Проверить статус добавлено раннера:
![Alt text](images/gitlab_runner_registration.png)

---

9) Осталось добавить в `.gitlab-ci.yml` ваш `runner`:
```yaml
  tags: 
    - name_of_runner
```
**CI настроен!**

---

# Лабораторная работа (проект)

В группах по 2 человека (в исключительных случаях 3) создать инфраструктуру для разработки (CI) и, используя готовую инфраструктуру, написать небольшое приложение. Во время работы использовать ["Git flow"](https://docs.github.com/en/get-started/quickstart/github-flow) (идея -> фича -> ветка -> merge request -> review -> master). Финальный релиз закрепить в "Realeses".

Цель лабораторной работы: продемонстрировать владение практическими навыками построения CI-инфраструктуры, навыки работы с Git, навыки совместной работы.

---

Требования:
- Завести репозиторий в Gitlab
- Настроить CI, отвечающий следующим требованиям
    - CI pipeline вида: 
    `linters (>=2) - builds (все платформы) - tests (>=2) - packaging (exe+libs / .jar)`
    - одним из линтеров должен быть самописный скрипт (python, bash, javascript), который проверяет copyright в начале исходных файлов проекта 
    - сборка на Windows и Linux
    - работа в docker-контейнерах
    - виды тестирования: юнит и функциональное
- Разработать консольный калькулятор (можно простой, без поддержки приоритета операций) на ЯП C++ или Java

---

Защита проекта, 7 минут:
- демонстрация работы инфраструктуры (~1.5 минуты);
- техническое ревью приложения (~0.5 минуты);
- техническое ревью инфраструктуры (~3 минуты);
- ревью процесса разработки (~2 минуты);
- ответы на вопросы (по проекту + теория).

Каждый член команды должен знать всё по компонентам проекта. На оценку влияют: степень готовности проекта, понимание проделанной работы и знание всей теории.

---
![Alt text](images/ci_error.png)

Если после конфигурирования `gitlab-ci.yml` у вас предупреждение вида "нам нужно валидировать ваш аккаунт", то не беда! Вы можете установить gitlab-runner на любой собственной машине. См. раздел "Установка и настройка gitlab-runner".

---

# До встречи!
