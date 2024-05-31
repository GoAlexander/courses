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

# Инфраструктура для разработки (или поговорим о совместной разработчке, инфраструктуре, CI и DevOps)
автор: Дыдычкин Александр Алексеевич
email: AlexanderDydychkin@yandex.ru

---

# Вступление

- Обсудим название
- Это не курс по DevOps?
- Курс актуален для аналитиков/разработчиков/инфраструктурщиков? 
- Курс содержит специфику Gitlab?

---

# Проблемы в процессе разработки ПО

---

![Alt text](images/process_cleaned.png)

---

![Alt text](images/process.png)

---

## А также сокращение издержек!

---

![Alt text](images/agile.webp)

---

### Agile manifest

> 1. Individuals and interactions over processes and tools
> 2. Working software over comprehensive documentation
> 3. Customer collaboration over contract negotiation
> 4. Responding to change over following a plan

---

### Как разрабатывается ПО?

---

![Alt text](images/dev_process.png)

---

Проблемы на ~~бочку~~ доску!

*(типичная проблема поставки / релиза...)*

---

# Решения проблем. CI. DevOps

---

## CI - если больно, то давайте делать ЭТО чаще!

- техническая ретроспектива: раньше был cron + .sh скрипты

---

![Alt text](images/ci.png)

---

## DevOps

- команда, а не человек
- сотрудничество команд разработки и эксплуатации
- навыки написания кода
- конвееры CI/CD используются активно
- все есть текст / "infrastructure as a code" (git)
- культура взаимодействия
- отлаженный процесс развертывания
- автоматизация рутины, ещё раз автоматизация, много автоматизации

---

## Разница между CI и DevOps?

- культура
- степень автоматизации

---

![Alt text](images/devops_trends.png)

---

## Ок. Проблемы порешали. А что нужно, чтобы построить инфраструктуру для разработки?

---

## Инструментарий
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
- all-in-one (представитель "2-ого поколения");
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

# Проектная (групповая) работа

**Общее описание:** В группах по 3 человека создать инфраструктуру для разработки (CI) и, используя инфраструктуру, написать приложение. Во время работы использовать ["Git flow"](https://docs.github.com/en/get-started/quickstart/github-flow) (идея -> фича -> ветка -> merge request -> review -> master). 

**Цель проектной работы:** продемонстрировать владение практическими навыками построения CI-инфраструктуры, навыки работы с Git, навыки совместной работы. Научиться работе "как в реальных компаниях", "как на реальной работе". Эмулировать настоящий рабочий процесс.

**Предупреждение:** данная работа - самая важная и сложная за весь курс. Ее невозможно сделать за "последние пару дней" и даже за неделю. Если хотите получить оценку выше удовлетворительно, то придется работать несколько недель, задавая на практиках конкретные технические вопросы.

---

Требования:
1) Завести репозиторий в Gitlab
2) Настроить CI, отвечающий следующим требованиям
    - CI pipeline вида: 
    `linters (>=2) - builds (2 Linux + 1 Windows) - tests (>=2) - packaging (exe+libs)`
    - одним из линтеров должен быть самописный скрипт (python, bash, javascript), который проверяет copyright в начале исходных файлов проекта
    - сборка на 2-ух Unix-based платформах и на 1 Windows (Windows на оценку 10)
    - работа в docker-контейнерах
    - виды тестирования: юнит (с помощью любого фреймворка) и use-case тестирование с помощью конвеера (pipes)
    - packaging (пакетирование)

---

**packaging (пакетирование)** - превращение бинарного исполняемого файла в продукт (нечто чем удобно пользоваться конечному пользователю). Примеры пакетирования: подготовить инсталлятор или архив со всеми зависимостями и документацией.

3) Разработать консольный калькулятор с поддержкой ввода выражений в виде параметров (можно без поддержки приоритета операций) на C++
4) Подготовить документацию (README.md) с разделами: для разработчика и для конечного пользователя
5) Выложить Dockerfile'ы использованных образов
6) Во время работы придерживаться Git flow. **Практика и понимание Git flow будет также оцениваться.**
7) Финальный релиз закрепить в "Releases".

Пример open source проекта с настроенной инфраструктурой и отработанными процессами: https://github.com/aristocratos/btop

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
