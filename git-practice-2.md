# Практика 2: Git + GitHub Pages (консоль, «с нуля до личной страницы»)

> Цель: установить Git, настроить SSH с GitHub, стянуть шаблон `matrix.html` с учебного сервера, персонализировать его **в консоли**, выложить как сайт на GitHub Pages.

---

## 0) Что понадобится

* Компьютер с Linux (или macOS; ниже отмечены отличия).
* Учётная запись GitHub (создать: [https://github.com/join](https://github.com/join)).
* Доступ в интернет, утилиты `git`, `ssh`, `curl`, `sed`.

Если инструкция лежит на сервере, её можно скачать:

```bash
curl -O http://51.250.15.155/git-practice-2.md
```

---

## 1) Установка Git и базовая конфигурация

**Linux (Debian/Ubuntu):**

```bash
sudo apt update && sudo apt install -y git
```

**macOS:**

```bash
brew install git
```

**Проверка и глобальные настройки:**

```bash
git --version
git config --global user.name  "Имя Фамилия"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

> Эти параметры попадут в метаданные коммитов. Используйте реальное имя/почту, связанную с GitHub.

---

## 2) Регистрация на GitHub и подготовка SSH-ключа

1. Зарегистрировать/войти: [https://github.com](https://github.com)
2. Сгенерировать SSH-ключ **Ed25519**:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
# Нажать Enter на путь (по умолчанию ~/.ssh/id_ed25519), при желании — поставить passphrase
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

3. Скопировать **публичный** ключ и добавить его в GitHub:

   * Показать ключ: `cat ~/.ssh/id_ed25519.pub`
   * Скопировать в буфер (Linux, если установлен xclip): `xclip -sel clip < ~/.ssh/id_ed25519.pub`
   * В GitHub: **Settings → SSH and GPG keys → New SSH key → вставить → Add SSH key**.

4. Проверить соединение:

```bash
ssh -T git@github.com
# Должно ответить: "Hi <USERNAME>! You've successfully authenticated, but GitHub does not provide shell access."
```

---

## 3) Создать репозиторий под GitHub Pages (user site)

В браузере на GitHub создайте **публичный** репозиторий с именем:

```
<ВАШ_ЛОГИН>.github.io
```

Примеры: `annpetrova.github.io`, `ivan-iv.github.io` и т.д.

> Это особый репозиторий: содержимое ветки `main` будет опубликовано как сайт.

---

## 4) Скачать шаблон страницы с сервера и подготовить файлы

Рабочая папка (любая, пример ниже):

```bash
mkdir -p ~/hse-git-site && cd ~/hse-git-site
```

Скачать файл **matrix.html**:

```bash
curl -O http://51.250.15.155/matrix.html
```

Переименовать в `index.html` (GitHub Pages ищет именно `index.html`):

```bash
mv matrix.html index.html
```

---

## 5) Персонализация HTML в **консоли**

Шаблон содержит строковые плейсхолдеры (например, `ILYA` и `IVANTSOV`).
Замените их на свои **имя** и **фамилию** (и при желании — `<title>` страницы).

### Вариант A — Linux (GNU sed)

```bash
export NAME="Ivan"
export SURNAME="Petrov"
export TITLE="My Matrix Page"

sed -i "s/ILYA/${NAME}/g; s/IVANTSOV/${SURNAME}/g; s/<title>[^<]*</<title>${TITLE}</" index.html
```

### Вариант B — macOS (BSD sed требует пустой «маски бэкапа»)

```bash
export NAME="Ivan"
export SURNAME="Petrov"
export TITLE="My Matrix Page"

sed -i '' "s/ILYA/${NAME}/g; s/IVANTSOV/${SURNAME}/g; s/<title>[^<]*</<title>${TITLE}</" index.html
```

Проверка, что плейсхолдеры исчезли:

```bash
grep -nE 'ILYA|IVANTSOV' index.html || echo "OK: персонализировано"
```

> Не хотите искать плейсхолдеры? Можно открыть файл просмотрщиком: `head -n 40 index.html` или `less index.html`.

---

## 6) Инициализировать Git-репозиторий и первый коммит

```bash
git init
touch .nojekyll               # для «голого» HTML без Jekyll
git add index.html .nojekyll
git commit -m "Initial personal page"
git branch -M main
```

---

## 7) Привязать удалённый репозиторий (SSH) и отправить

Замените `USERNAME` на свой логин GitHub:

```bash
git remote add origin git@github.com:USERNAME/USERNAME.github.io.git
git push -u origin main
```

---

## 8) Включить публикацию GitHub Pages (если не включилась автоматически)

В репозитории на GitHub → **Settings → Pages**:

* **Build and deployment**: **Source: Deploy from a branch**
* **Branch**: `main`  / **Folder**: `/`

Через 1–2 минуты сайт будет доступен по адресу:

```
https://USERNAME.github.io/
```

Проверка из консоли:

```bash
curl -I https://USERNAME.github.io/
```

---

## 9) Готово. Что показать преподавателю

* Ссылка на сайт: `https://USERNAME.github.io/`
* Вывод `git log --oneline --graph --decorate -n 3`
* Скрин/вывод `ssh -T git@github.com` с «Hi USERNAME! …»

---

## (Опционально) Скрипт «всё сразу»

> Используйте **только** если уже создали репозиторий `USERNAME.github.io` и добавили SSH-ключ в GitHub.

```bash
# ПОДСТАВИТЕ СВОИ ДАННЫЕ:
GHUSER="your_github_username"
GHEMAIL="you@example.com"
NAME="Ivan"
SURNAME="Petrov"
TITLE="My Matrix Page"

# SSH-ключ (создастся, если его нет)
[ -f ~/.ssh/id_ed25519 ] || ssh-keygen -t ed25519 -C "$GHEMAIL" -N "" -f ~/.ssh/id_ed25519
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519

# Проект
mkdir -p ~/hse-git-site && cd ~/hse-git-site
curl -fsSLO http://51.250.15.155/matrix.html
mv matrix.html index.html

# Кроссплатформенный sed (Linux/macOS)
if sed --version >/dev/null 2>&1; then SED="sed -i"; else SED="sed -i ''"; fi
$SED "s/ILYA/${NAME}/g; s/IVANTSOV/${SURNAME}/g; s/<title>[^<]*</<title>${TITLE}</" index.html

git init
git config user.name  "$NAME $SURNAME"
git config user.email "$GHEMAIL"
git config init.defaultBranch main
touch .nojekyll
git add index.html .nojekyll
git commit -m "My page"
git branch -M main
git remote add origin git@github.com:$GHUSER/$GHUSER.github.io.git
git push -u origin main

echo "Откройте сайт: https://$GHUSER.github.io/"
```

---

## Частые ошибки и быстрые решения

* **`Permission denied (publickey)` при `git push`**
  → SSH-ключ не добавлен в GitHub или не загружен в агент.
  Проверьте: `ssh -T git@github.com`. Если не «Hi USERNAME…», повторите раздел 2.

* **`Repository not found` при `git push`**
  → Репозиторий не создан или имя не совпадает (`USERNAME.github.io`). Создайте/проверьте.

* **`sed: illegal option --` на macOS**
  → Используйте синтаксис `sed -i '' "s/OLD/NEW/" file` (см. раздел 5, вариант B).

* **Страница не открывается через пару минут**
  → Проверьте **Settings → Pages** (источник — `Deploy from a branch`, ветка `main`, папка `/`).
  Убедитесь, что файл называется **`index.html`** и лежит в корне репозитория.

---

## Полезные команды

```bash
git status
git log --oneline --graph --decorate --all
git show HEAD:index.html | sed -n '1,40p'
```

---

### Итог

* Git установлен и настроен.
* SSH-связка с GitHub проверена.
* Персонализированный `index.html` отправлен в `USERNAME.github.io`.
* Сайт доступен по `https://USERNAME.github.io/`.

