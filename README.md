# BookLove — Legal pages (Privacy Policy & Terms of Use)

Статические страницы для GitHub Pages: `index.html`, `privacy.html`,
`terms.html`, `support.html` + `assets/style.css`.

Живой адрес: **https://volynetsvitalii.github.io/booklove-legal/**
(репозиторий `github.com:VolynetsVitalii/booklove-legal.git`; локально лежит
в `LegalPages/` как ОТДЕЛЬНЫЙ git-репозиторий).

---

## Universal Links — как включить (T-200 → T-230)

В T-200 построен весь слой ссылок, но САМИ Universal Links пока выключены.
Ниже — что уже готово и что нужно сделать, чтобы их включить.

### Что уже сделано (T-200)

* **`.well-known/apple-app-site-association`** — файл в этой папке. Формат
  современный (`components`), пути: `/b/*` (книга по ISBN), `/wrapped/*`
  (карточка-обёртка), `/club/*` (приглашение в клуб).
  `appIDs` — `8MZT9FWMM3.com.vitaliivolynets.BookLove` (Team ID из
  `fastlane/Appfile` + bundle id).
* **Приложение уже понимает эти адреса.** `DeepLinkParser` принимает и
  `https://booklove.app/b/<isbn>`, и `https://volynetsvitalii.github.io/booklove-legal/b/<isbn>`,
  и собственную схему `booklove://b/<isbn>`. Схема `booklove` объявлена в
  `BookLove/Info.plist` (`CFBundleURLTypes`) — благодаря этому deep-link'и
  можно проверять уже сейчас:
  ```bash
  xcrun simctl openurl booted "booklove://b/9780441013593"
  ```

### Что осталось (три шага, все в T-230)

1. **Положить AASA в КОРЕНЬ домена.** iOS запрашивает файл строго по адресу
   `https://<домен>/.well-known/apple-app-site-association` — путь домена
   значения не имеет, подпапка НЕ подойдёт.
   Практическое следствие: на текущем project-репозитории GitHub Pages
   (`volynetsvitalii.github.io/booklove-legal/`) Universal Links включить
   НЕЛЬЗЯ. Нужен либо репозиторий пользовательского сайта
   (`volynetsvitalii.github.io`), либо собственный домен `booklove.app`
   из T-230. Файл должен отдаваться по HTTPS, с `Content-Type:
   application/json`, без редиректов и без подписи.
   Проверка после публикации:
   ```bash
   curl -sI https://booklove.app/.well-known/apple-app-site-association
   ```

2. **Включить capability и entitlement.** В Apple Developer Portal включить
   Associated Domains для App ID, перевыпустить профили, затем добавить в
   `BookLove/BookLove.entitlements`:
   ```xml
   <key>com.apple.developer.associated-domains</key>
   <array>
     <string>applinks:booklove.app</string>
   </array>
   ```
   ⚠️ Порядок важен: entitlement БЕЗ включённой capability ломает подпись, и
   релизная сборка fastlane падает. Именно поэтому в T-200 его не добавляли.

3. **Переключить конфигурацию в приложении.** В
   `BookLove/Features/Share/Models/ShareLinkConfiguration.swift`:
   ```swift
   static let current = ShareLinkConfiguration(
       webBase: futureDomain,          // https://booklove.app
       appStoreID: "6761546582",
       isUniversalLinkingLive: true    // было false
   )
   ```
   После этого кнопки «Поделиться» начнут раздавать ссылки вида
   `booklove.app/b/<isbn>?utm_source=…` вместо ссылки App Store с меткой `ct`.

### Пока Universal Links выключены

Ссылки ведут на страницу приложения в App Store с campaign-токеном:
`https://apps.apple.com/app/id6761546582?ct=book_detail&mt=8`.
Это работающая атрибуция без своего сервера: разбивку по `ct` показывает
Apple в App Store Connect → Analytics → Acquisition → Campaigns.
