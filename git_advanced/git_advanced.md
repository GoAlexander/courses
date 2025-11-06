---
marp: true
author: Alexander Dydychkin
title: Git. Advanced.
theme: default
paginate: true
math: mathjax
footer: "Star slides [here](https://github.com/GoAlexander/courses)"

---

# Git. Продвинутый уровень.

---

## Подразумевается, что учащийся уже знает:
- что такое система контроля версий (VCS) и зачем они нужны
- особенности git от остальных VCS
- что такое хостинги кода (github, gitlab и т.д.)
- git и github (gitlab) flows ([тыц1](https://www.atlassian.com/git/tutorials/comparing-workflows), [тыц2](https://habr.com/ru/articles/767424/), [тыц3](https://konstantinklepikov.github.io/myknowlegebase/notes/workflow-types.html))
- принцип атомарности операций, хеш (commit id), sha1
- состояния в git (untracked, staged, committed, modified). [Тыц.](http://ndpsoftware.com/git-cheatsheet.html#loc=index;)
- ветвление, remote, default branch (master/main), commit message
- основные **команды**: `init`, `clone`, `pull`, `add`, `commit`, `push`, `branch`, `checkout`, `status`, `diff`.

[Для освоения начального уровня можете пройти эти слайды.](https://github.com/GoAlexander/courses/blob/ddbf010ca822ff7f5873e66c86e09fdce95de468/git_advanced/git_beginner_edited_2023.pdf)

---

## Редактирование последнего коммита

> Use-case: забыли добавить какой-то файл/изменение в коммит или нужно отредактировать **commit message**.

```bash
git add files
git commit --amend
```

---

## Редактирование истории

> Use-case 1: squash, много "мусорных" коммитов, нужно соединить их в один осмысленный.
> Use-case 2: edit, во время ревью просят отредактировать историю.
```bash
git rebase -i HEAD~10
# или
git rebase -i sha
```
- squash
- edit previous commits

---

## Если ну очень нужно запушить (пуш ветки с отредактированной историей)
> Use-case 1: нужно получить/обновить чью-то dev ветку после использования `git rebase`.
```bash
git push origin branch --force

# обратная операция (получения ветки с отредактированной историей)
git pull origin branch --force
```

Методы защиты:
- protected branches
- PR/MR, review

---

## Stash (временное хранилище, "тайник")
> Use-case 1: нужно временно "убрать" не закрепленные изменения к примеру перед `git rebase`.
```bash
git stash # сохранение
git stash pop # получение
```
Работает как стэк.

---

## Работа с несколькими ветками одновременно в одном репозитории
> Use-case 1: нужно разрабатывать две фичи *(=две ветки)* **одновременно** в одном репозитории.

```bash
git worktree add ../feature_x feature_x # папка, ветка
```
Профит:
- экономия на `.git`

---

## Перенос коммита

```bash
git cherry-pick sha
```

---

## Патчи

> Use-case 1: по легальным, архитектурным или бизнес причинам иногда нужно распространять open source тулу + кастомные патчи к ней.

```bash
git diff > patch.diff # создание диффа из незафиксированных изменений
git format-patch HEAD~1 # создание патча из последнего коммита
git apply file
```

---

## Откат ветки

```bash
git reset --hard sha # откат к указанному состоянию и "удаление" всех текущих изменений
```

---

## Git files

- `.git`
- `.gitignore`
- `.gitattributes`

---

## Хуки

> Use-case 1: нужно автоматизировать некоторую проверку перед созданием коммита.

`git hooks` - благодаря этому механизму можно привязать некие действия к коммиту. Могут быть запущены на клиенте, сервере, перед созданием коммита, после и т.д

[больше чтения](https://git-scm.com/book/ru/v2/%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-Git-%D0%A5%D1%83%D0%BA%D0%B8-%D0%B2-Git)

---

## ?

```bash
git commit -p file # к примеру позволяет закоммитить только часть файла
```

Multiple remotes:
```bash
git remote add "origin" git@github.com:User/UserRepo.git
git remote set-url "origin" git@github.com:User/UserRepo.git
git push origin
```

**Use git over SSH!**
- https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
- https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account

---

## Типы репозиториев

- обычные репозитории
- монорепозиторий (к примеру используется в AOSP, яндекс)

---

## "Extensions"

- `git submodule` - благодаря этому механизму можно в репе X сослаться на состояние репозитория Y (в некотором смысле ответ монорепам).
*НЕ НУЖНО!*

- `git lfs` - иногда к репозиторию нужно привязать бинарные объекты. Не является частью традиционного репозитория.

---

## ???

```bash
git bisect # помогает локализовать коммит, в котором появился ошибка X

git blame # кто, что и где редактировал в этом файле?
```

---

## IT'S TIME TO GO DEEEEEPER
### ref-log
> Use-case 1: был выполнен неправильный `reset --hard` или неправильный `rebase`. Нужно откатить состояние.

В git (почти) всё сохраняется! Пока не заработает сборщик мусора...

```bash
git ref-log
```

`ref-log` показывает состояния, в которых был ваш **локальный** репозиторий. Условно история для `ctrl+z`.

---

> The reflog is an ordered list of the commits that HEAD has pointed to: it's undo history for your repo. The reflog isn't part of the repo itself (it's stored separately to the commits themselves) and isn't included in pushes, fetches or clones; it's purely local.

> *Aside: understanding the reflog means you can't really lose data from your repo once it's been committed. If you accidentally reset to an older commit, or rebase wrongly, or any other operation that visually "removes" commits, you can use the reflog to see where you were before and `git reset --hard` back to that ref to restore your previous state. Remember, refs imply not just the commit but the entire history behind it.*

[Source](https://stackoverflow.com/a/17860179)

---

### Сантехника и фарфор... (тезисно)
- c точки зрения структуры коммитов git - ориентированный ациклический граф (DAG);
    - child -> (multiple) parents
    - 🤔как коммит ссылается на другой коммит?
- git оперирует `snapshot`'ами, слепками ФС [(чуть больше)](https://t.me/oom_ru/103)
- `snapshot`'ы обеспечивают скорость, но жутко избыточны
- борьба с избыточностью ведётся на уровне `blob`'ов (Binary Large Object) и частично `pack`'ов

---

## Бонус: GUI

- Sublime Merge 
- VS Code + `mhutchie.git-graph`
