---
title: نصب ویرایشگر کد
description: مروری بر Sublime Text و شیوه پیکربندی آن
icon: material/text-box-edit
---

# نصب ویرایشگر کد

اولین قدم این است که یک ویرایشگر کد برای مدیریت فایل‌های Elm راه‌اندازی کنید.

![text-editor](../assets/images/editor.webp)

تعدادی افزونه توسط اعضای جامعه کاربری برای طیف گسترده‌ای از ویرایشگرها نگهداری می‌شود. فهرستی از آن‌ها را می‌توانید در صفحه [افزونه‌های ویرایشگر][editor-plugins]{: .external } مشاهده کنید.

راه‌اندازی یک ویرایشگر کد می‌تواند دشوار باشد، بنابراین شیوه راه‌اندازی Sublime Text را نشان می‌دهم. امیدوارم این شیوه، برای برنامه‌نویسان تازه کار و حرفه‌ای، مفید باشد.

## Sublime Text

**گام اول:** Sublime Text را از [وبسایت رسمی][sublime-text]{: .external } دانلود کنید.

**گام دوم:** افزونه "Elm Syntax Highlighting" را نصب کنید.

- [ویندوز](https://github.com/evancz/elm-syntax-highlighting/blob/master/install/windows.md){: .external }
- [مک اواس](https://github.com/evancz/elm-syntax-highlighting/blob/master/install/mac.md){: .external }
- [لینوکس](https://github.com/evancz/elm-syntax-highlighting/blob/master/install/linux.md){: .external }

پس از گذراندن این مراحل، باید بتوانید فایل‌های Elm را با Syntax Highlighting باز کنید. کلمات کلیدی مانند `import` و `type` باید رنگی باشند تا خواندن کد آسان‌تر شود.

با اتمام فرآیند نصب `elm`، دسترسی از طریق ترمینال و تنظیم ویرایشگر کد، بیایید به یادگیری Elm ادامه دهیم!

!!! note "یادداشت"

	گزینه‌های دیگری هم وجود دارد! اعضای جامعه کاربری، افزونه‌های دیگری برای ویرایشگرهای Emacs، IntelliJ، Vim، VS Code و بسیاری دیگر ایجاد کرده‌اند. سعی می‌کنیم صفحه [افزونه‌های ویرایشگر][editor-plugins]{: .external } را با تمام گزینه‌های موجود، بروز نگه داریم!

***

!!! abstract "یادداشت مترجم"

	ویرایشگرهای کد قابلیتی به نام **Language Server** دارند که برای هر زبان برنامه‌نویسی، پیاده‌سازی جداگانه‌‌ای دارد. این ابزار در جامعه کاربری **Elm** با نام `elm-language-server` شناخته می‌شود که یکی دیگر از پروژه‌های `elm-tooling` به حساب می‌آید. پلاگین موجود در **VSCode** شامل این ابزار می‌شود، اما سایر ویرایشگرهای کد باید آن را به صورت جداگانه نصب و پیکربندی کنند. برای کسب اطلاعات بیشتر، به منابع زیر مراجعه کنید:

	- [Elm Language Server][elm-language-server]{: .external } 
	- [VSCode Plugin][elm-ls-vscode]{: .external }

[editor-plugins]: https://github.com/elm/editor-plugins
[sublime-text]: https://www.sublimetext.com
[elm-language-server]: https://github.com/elm-tooling/elm-language-server
[elm-ls-vscode]: https://marketplace.visualstudio.com/items?itemName=Elmtooling.elm-ls-vscode