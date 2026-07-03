# BookLove — Legal pages (Privacy Policy & Terms of Use)

Статические страницы для App Store (**T-143**). Хостятся бесплатно на **GitHub
Pages**. Ссылки на них уже встроены в приложение (Settings → About и на Paywall)
и указываются в App Store Connect.

```
LegalPages/
├── index.html        # Титульная: ссылки на обе страницы
├── privacy.html      # Privacy Policy (App Store требует именно её URL)
├── terms.html        # Terms of Use (EULA-дополнение)
├── assets/
│   └── style.css     # Общий стиль (system font, dark mode, без внешних ресурсов)
└── README.md         # Этот файл
```

Никаких внешних шрифтов, скриптов или картинок — страницы самодостаточны,
грузятся мгновенно и не «сливают» данные посетителя (privacy даже в вёрстке).

---

## Как опубликовать на GitHub Pages (5 минут, бесплатно)

Рекомендуемый способ — **отдельный публичный репозиторий** `booklove-legal`.
Так основной репозиторий приложения может оставаться приватным, а юридические
страницы (которые обязаны быть публичными) живут отдельно.

1. Создай на GitHub **новый публичный репозиторий** с именем `booklove-legal`.
2. Скопируй в его корень **содержимое** этой папки (`index.html`, `privacy.html`,
   `terms.html`, папку `assets/`). Важно: файлы должны лежать в корне репозитория,
   а не внутри вложенной папки `LegalPages/`.
3. Запушь и открой на GitHub: **Settings → Pages**.
4. В разделе *Build and deployment* выбери **Source: Deploy from a branch**,
   ветка `main`, папка `/ (root)`. Нажми **Save**.
5. Через ~1 минуту сайт будет доступен по адресу:

   ```
   https://volynetsv.github.io/booklove-legal/
   ├── privacy.html   → https://volynetsv.github.io/booklove-legal/privacy.html
   └── terms.html     → https://volynetsv.github.io/booklove-legal/terms.html
   ```

> **Замени `volynetsv` на свой реальный GitHub-логин** (в URL он всегда в нижнем
> регистре). Если возьмёшь другое имя репозитория — поправь и URL-ы в коде
> приложения: константы `privacyPolicyURL` и `termsOfUseURL` в
> `BookLove/Features/Settings/Models/AppInfo.swift` (это **единственный** источник
> правды — их используют и Settings, и Paywall).

Хочешь короткий адрес вида `booklove.app/privacy`? Купи домен и добавь файл
`CNAME` в репозиторий (GitHub → Settings → Pages → Custom domain). Для запуска
это не обязательно — адрес `github.io` полностью подходит для App Store.

---

## Куда вставить ссылки в App Store Connect

App Store Connect требует **URL Privacy Policy** обязательно, а Terms — по желанию
(иначе действует стандартный Apple EULA).

- **Privacy Policy URL** (обязательно):
  App Store Connect → твоё приложение → вкладка **App Privacy** →
  *Privacy Policy* → вставить `https://volynetsv.github.io/booklove-legal/privacy.html`.
- **Terms of Use (EULA)** (опционально):
  App Store Connect → **App Information** → *License Agreement*. Оставь стандартный
  Apple EULA **или** укажи кастомный. Ссылку на страницу Terms также добавляют в
  поле описания/маркетинга при необходимости.

Внутри приложения обе ссылки уже открываются из **Settings → About** и со **страницы
подписки (Paywall)** — код менять не нужно, только константы URL (см. выше), если
поменяешь логин/репозиторий.

---

## Важное про содержание (сверено с кодом — security-review, T-143)

Privacy Policy написана **честно и полно**, а не по упрощённой формулировке
«данные не собираются вообще». Проверка кода показала, что за пределы устройства
уходит ограниченный набор данных, и он **раскрыт** в политике:

| Что уходит с устройства | Куда | Раздел в privacy.html |
|---|---|---|
| Поисковый запрос / отсканированный ISBN | Google Books, Open Library | §6 |
| Вся библиотека (только твой приватный iCloud) | Apple CloudKit **private** | §5 |
| Анонимные события (без названий книг/ID) | TelemetryDeck | §8 |
| Статус покупки + анонимный ID | RevenueCat / Apple | §9 |

Кадры с камеры обрабатываются **на устройстве** (сканер ISBN) и не загружаются —
это тоже отражено (§7) и совпадает с текстом `NSCameraUsageDescription` в Info.plist.

> Файл `PrivacyInfo.xcprivacy` (Privacy Manifest / «этикетка данных» и Required
> Reason API) — это задача **T-144**, здесь мы его сознательно не трогаем.
