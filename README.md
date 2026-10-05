# BooksLove - сайт

Статический сайт BooksLove на GitHub Pages: лендинг, SEO-страницы,
юридические документы, пресс-кит и роутер входящих deep-link'ов.

Живой адрес: **https://volynetsvitalii.github.io/booklove-legal/**
(репозиторий `github.com:VolynetsVitalii/booklove-legal.git`; локально лежит
в `LegalPages/` как ОТДЕЛЬНЫЙ git-репозиторий внутри репозитория приложения).

---

## Структура

| Файл | Что это |
|---|---|
| `index.html` | **Лендинг.** Hero, пять секций-фич, блок переезда с Goodreads, манифест приватности, FAQ, призыв. |
| `goodreads-alternative.html` | SEO-страница под запрос «goodreads alternative»: честное сравнение + раздел «чего у нас нет». |
| `import-from-goodreads.html` | SEO-страница-инструкция: шесть шагов переезда, разметка `HowTo`. |
| `reading-tracker.html` | SEO-страница категории: шесть способов вести учёт чтения. |
| `wrapped.html` | Посадочная страница для ссылок `/wrapped/*`. |
| `changelog.html` | Журнал версий. |
| `press.html` | Пресс-кит: факты, три готовых описания, скриншоты, контакт. |
| `legal.html` | Хаб юридических документов (до T-230 был `index.html`). |
| `privacy.html`, `terms.html`, `support.html` | Юридические документы. |
| `404.html` | **Роутер** `/b/*`, `/wrapped/*`, `/club/*` + обычная страница «не найдено». |
| `robots.txt`, `sitemap.xml` | Для поисковых систем. |
| `.well-known/apple-app-site-association` | Файл Universal Links (см. ниже). |

### Оформление

Два стилевых слоя, и это сделано намеренно:

* `assets/landing.css` - маркетинговые страницы. Широкая сетка, крупная
  засечная типографика, рамки телефонов, появление секций при прокрутке.
* `assets/style.css` - юридические документы. Узкая колонка, ноль украшений,
  максимальная читаемость.

Один файл на оба случая пришлось бы всё время переопределять, и каждая правка
лендинга рисковала бы сдвинуть Privacy Policy.

**Ноль внешних запросов.** Ни шрифтов с CDN, ни аналитики, ни встроенных
видео, ни бейджа App Store с серверов Apple. Причина не в скорости: приложение
обещает «no cross-app tracking» (T-394), и сайт этого приложения не может
сообщать чужому серверу IP-адрес каждого, кто открыл страницу с этим обещанием.

### Картинки

Пересобираются из репозитория приложения:

```bash
# Скриншоты: из fastlane/screenshots, ужать до 640 px и перевести в JPEG
for src in "fastlane/screenshots/en-US/iPhone 17 Pro Max-"*.png; do
  name=$(basename "$src" .png); name="${name#iPhone 17 Pro Max-}"
  slug=$(echo "$name" | sed 's/^[0-9][0-9]//' | tr '[:upper:]' '[:lower:]')
  cp "$src" "LegalPages/assets/screens/$slug.png"
  sips --resampleWidth 640 "LegalPages/assets/screens/$slug.png" >/dev/null
  sips -s format jpeg -s formatOptions 82 "LegalPages/assets/screens/$slug.png" \
       --out "LegalPages/assets/screens/$slug.jpg" >/dev/null
  rm "LegalPages/assets/screens/$slug.png"
done

# Иконка сайта
cp BookLove/Resources/Assets.xcassets/AppIcon.appiconset/AppIcon1024.png LegalPages/assets/icon.png
sips --resampleWidth 180 LegalPages/assets/icon.png >/dev/null

# Картинка превью для мессенджеров (1200×630)
swift Scripts/og-image.swift
```

### Бейдж App Store

`assets/appstore-badge.svg` нарисован вручную, чтобы не тянуть картинку с
серверов Apple. Apple разрешает и предпочитает свою оригинальную графику:
скачать её можно на
`developer.apple.com/app-store/marketing/guidelines/` и положить в этот же
файл - пропорции 3:1 те же, правок в CSS и в разметке не потребуется.

---

## Сторож

Соответствие сайта приложению проверяет `Scripts/landing-audit.sh` из
репозитория приложения (шаг 6г в `fastlane ios preflight`):

```bash
./Scripts/landing-audit.sh
```

Семь проверок: мета-теги на каждой странице · Smart App Banner и app-id против
`ShareLinkConfiguration` · целостность локальных ссылок · отсутствие внешних
запросов · сверка `sitemap.xml` с набором страниц в обе стороны · валидность
AASA и совпадение маршрутов с `DeepLinkParser` · отсутствие непроверяемых
заявлений и обещаний невыпущенных функций.

Скрипт спокойно переживает отсутствие папки `LegalPages` (сайт - отдельный
репозиторий): предупреждает и выходит с нулём.

### Локальный просмотр

```bash
cd LegalPages && python3 -m http.server 8765
open http://localhost:8765/
```

Роутер проверяется так же - `http://localhost:8765/b/9780441013593`. Обычный
`http.server` вернёт код 404 без тела `404.html`; GitHub Pages при том же коде
отдаёт содержимое страницы. Чтобы увидеть роутер локально, откройте
`http://localhost:8765/404.html` - блок «не найдено» покажется потому, что в
адресе нет маршрута.

---

## Universal Links

Слой ссылок построен в T-200, сайт и роутер - в T-230. Сами Universal Links
**пока выключены**, и ниже - почему и что нужно, чтобы их включить.

### Что уже готово

* **`.well-known/apple-app-site-association`** - лежит в этой папке. Формат
  современный (`components`), маршруты `/b/*`, `/wrapped/*`, `/club/*`,
  `appIDs` - `8MZT9FWMM3.com.vitaliivolynets.BookLove`. Валидность файла и
  совпадение маршрутов с `DeepLinkParser` проверяет сторож (проверка F).
* **Приложение уже понимает эти адреса.** `DeepLinkParser` принимает
  `https://booklove.app/b/<isbn>`,
  `https://volynetsvitalii.github.io/booklove-legal/b/<isbn>` и собственную
  схему `booklove://b/<isbn>`. Схема объявлена в `BookLove/Info.plist`, поэтому
  маршрут проверяется уже сейчас, без домена и без AASA:
  ```bash
  xcrun simctl openurl booted "booklove://b/9780441013593"
  ```
* **Сайт уже обслуживает эти адреса** для человека БЕЗ приложения -
  роутер `404.html`.

### Почему выключены

iOS запрашивает AASA строго по адресу
`https://<домен>/.well-known/apple-app-site-association`. Путь домена значения
не имеет - **подпапка не подойдёт**. Текущий сайт живёт по адресу
`volynetsvitalii.github.io/booklove-legal/`, то есть в подпапке
project-репозитория GitHub Pages, и включить Universal Links там физически
нельзя.

Нужен корень домена: либо собственный `booklove.app`, либо репозиторий
пользовательского сайта `VolynetsVitalii.github.io`.

### Как включить, когда домен появится

Порядок важен. Шаг 3 без шага 2 ломает подпись релизной сборки - именно
поэтому entitlement не добавили ещё в T-200.

**1. Положить сайт в корень домена.**

Для собственного домена: файл `CNAME` с одной строкой `booklove.app` в корне
репозитория, A-записи GitHub Pages у регистратора, затем в настройках
репозитория - Pages → Custom domain и галочка Enforce HTTPS.

> Файл `CNAME` в репозитории **не заведён заранее** сознательно: пустой или
> указывающий на несуществующий домен `CNAME` немедленно обрушивает GitHub
> Pages целиком.

Проверка после публикации - файл обязан отдаваться по HTTPS, с
`Content-Type: application/json`, без редиректов и без подписи:

```bash
curl -sI https://booklove.app/.well-known/apple-app-site-association
# ожидается: HTTP/2 200 и content-type: application/json
```

**2. Включить capability в Apple Developer Portal.**
Certificates, Identifiers & Profiles → Identifiers → `com.vitaliivolynets.BookLove`
→ отметить **Associated Domains** → сохранить → **перевыпустить профили**
(иначе профиль на машине останется старым, без нового права).

**3. Добавить entitlement.** В `BookLove/BookLove.entitlements`:

```xml
<key>com.apple.developer.associated-domains</key>
<array>
  <string>applinks:booklove.app</string>
</array>
```

⚠️ Entitlement БЕЗ включённой capability ломает подпись, и `fastlane ios release`
падает на архивации.

**4. Переключить флаг в приложении.** В
`BookLove/Features/Share/Models/ShareLinkConfiguration.swift`:

```swift
static let current = ShareLinkConfiguration(
    webBase: futureDomain,          // https://booklove.app
    appStoreID: "6761546582",
    isUniversalLinkingLive: true    // было false
)
```

**5. Обновить адреса на сайте.** Заменить
`https://volynetsvitalii.github.io/booklove-legal/` на `https://booklove.app/`
в `canonical`, `og:url`, `sitemap.xml`, `robots.txt`, и очистить `BASE` в
роутере `404.html` (там же абсолютные пути `/booklove-legal/...` в `<head>` и
в разметке). После этого прогнать `./Scripts/landing-audit.sh`.

**6. Проверить на устройстве.** Universal Links **не работают в симуляторе
надёжно** и не срабатывают при вводе адреса прямо в адресную строку Safari -
нужно нажать на ссылку в Заметках или в сообщении. Первый запрос AASA система
делает при установке приложения; после правки файла на сервере переустановите
приложение.

### Пока выключены

Кнопки «Поделиться» раздают ссылку на App Store с меткой кампании:
`https://apps.apple.com/app/id6761546582?ct=book_detail&mt=8`. Это работающая
атрибуция без своего сервера и без единого трекера - разбивку по `ct` показывает
Apple в App Store Connect → Analytics → Acquisition → Campaigns. Те же метки
(`web_hero`, `web_nav`, `web_footer`, `web_link_book`, …) стоят на всех кнопках
сайта, поэтому видно, какая именно страница и какая кнопка приносят установки.
