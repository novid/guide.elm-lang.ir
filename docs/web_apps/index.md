---
title: وب اپلیکیشن
description: مقدمه‌ای بر وب اپلیکیشن، کنترل و میزبانی صفحه در Elm
icon: material/web
---

# وب اپلیکیشن

تا این فصل، برنامه‌های Elm را با استفاده از `Browser.element` ایجاد کرده‌ایم که به ما اجازه می‌دهد یک DOM Node را در یک برنامه بزرگ‌تر کنترل کنیم. این کار برای _معرفی_ Elm در محل کار عالی است (همانطور که در [این مقاله][elm-at-work]{: .external } توضیح داده شده است) اما بعد از آن چه اتفاقی می‌افتد؟ چگونه می‌توانیم از Elm بطور گسترده‌تری استفاده کنیم؟

در این فصل، یاد می‌گیریم که چگونه یک "وب اپلیکیشن" با تعدادی صفحه مختلف ایجاد کنیم که همه به خوبی با یکدیگر کار کنند، اما باید با کنترل یک صفحه واحد شروع کنیم.

## کنترل صفحه {#control-the-document}

اولین قدم این است که برنامه را با [`Browser.document`][Browser.document]{: .external } آغاز کنیم:

```elm
document :
  { init : flags -> ( model, Cmd msg )
  , view : model -> Document msg
  , update : msg -> model -> ( model, Cmd msg )
  , subscriptions : model -> Sub msg
  }
  -> Program flags model msg
```

آرگومان‌ها، بجز تابع `view`، تقریبا مشابه `Browser.element` هستند. به جای بازگشت یک مقدار `Html`، یک [`Document`][document]{: .external } به این شکل باز می‌گردانید:

```elm
type alias Document msg =
  { title : String
  , body : List (Html msg)
  }
```

این کار باعث می‌شود قابلیت کنترل `<title>` و `<body>` صفحه فعلی را داشته باشید. شاید برنامه مقداری داده دانلود کند تا هر صفحه عنوان سفارشی خود را داشته باشد. اکنون می‌توانید به سادگی این عنوان را در تابع `view` تغییر دهید!

## میزبانی صفحه {#serve-the-page}

کامپایلر بطور پیشفرض یک فایل HTML تولید می‌کند، بنابراین می‌توانید برنامه را به این شکل کامپایل کنید:

```bash
elm make src/Main.elm
```

خروجی یک فایل `index.html` خواهد بود که می‌توانید آن را مانند هر فایل HTML دیگری استفاده کنید. این کار به خوبی انجام می‌شود، اما می‌توانید با (۱) کامپایل Elm به جاوااسکریپت و (۲) ایجاد یک فایل سفارشی، انعطاف‌پذیری بیشتری بدست آورید. برای این منظور، برنامه را به این شکل کامپایل کنید:

```bash
elm make src/Main.elm --output=main.js
```

این دستور، فایل `main.js` را تولید می‌کند که می‌توانید از یک فایل HTML سفارشی آن را فراخوانی کنید:

```html linenums="1"
<!DOCTYPE HTML>
<html>
<head>
  <meta charset="UTF-8">
  <title>Main</title>
  <link rel="stylesheet" href="whatever-you-want.css">
  <script src="main.js"></script>
</head>
<body>
  <script>var app = Elm.Main.init();</script>
</body>
</html>
```

این فایل HTML نسبتا ساده است. هر چیزی که نیاز دارید را در قسمت `<head>` قرار می‌دهید و برنامه Elm را در قسمت `<body>` راه‌اندازی می‌کنید. برنامه Elm از آنجا ادامه می‌دهد و همه چیز را پردازش می‌کند.

در هر صورت، اکنون یک فایل HTML دارید که می‌توانید به مرورگر وب ارسال کنید. این فایل را می‌توانید با خدمات رایگانی مانند [GitHub Pages][github-pages]{: .external } یا [Netlify][netlify]{: .external } بدست کاربران برسانید، یا شاید یک سرور مجازی با خدماتی مانند [Digital Ocean][digital-ocean]{: .external } راه‌اندازی کنید. هر سرویس میزبانی که برای شما مناسب باشد! فقط به روشی نیاز دارید که فایل HTML به مرورگر وب کاربران ارسال شود.

!!! tip "نکته"

	اگر در حال سفارشی‌سازی با CSS هستید، ایجاد فایل HTML سفارشی مفید است. بسیاری از توسعه‌دهندگان از پروژه‌هایی مانند [`rtfeldman/elm-css`][elm-css]{: .external } برای مدیریت تمام استایل‌های خود از داخل Elm استفاده می‌کنند، اما شاید در تیمی کار می‌کنید که CSS از پیش تعریف شده دارد. شاید در تیمی کار می‌کنید که یکی از آن پیش‌پردازنده‌های CSS را استفاده می‌کند. ایرادی ندارد. فقط فایل CSS نهایی را در قسمت `<head>` فایل HTML قرار دهید.

	لینک Digital Ocean موجود در این صفحه، یک پیوند ارجاعی است، بنابراین اگر از طریق آن ثبت نام و از خدماتش استفاده کنید، یک اعتبار ۲۵ دلاری برای هزینه‌های میزبانی وبسایت‌های `elm-lang.org` و `package.elm-lang.org` دریافت خواهیم کرد.

*[DOM]: Document Object Model
*[HTML]: Hypertext Markup Language
*[CSS]: Cascading Style Sheets

[elm-at-work]: https://elm-lang.org/blog/how-to-use-elm-at-work
[Browser.document]: https://package.elm-lang.org/packages/elm/browser/latest/Browser#document
[document]: https://package.elm-lang.org/packages/elm/browser/latest/Browser#Document
[github-pages]: https://pages.github.com
[netlify]: https://www.netlify.com
[digital-ocean]: https://m.do.co/c/c47faa1916d2
[elm-css]: https://package.elm-lang.org/packages/rtfeldman/elm-css/latest