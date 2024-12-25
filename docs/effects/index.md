# دستور و اشتراک

در فصل‌های قبل، دیدیم که معماری Elm چگونه تعامل با ماوس و کیبورد را مدیریت می‌کند، اما درباره ارتباط با سرور یا تولید مقدار تصادفی چطور؟

برای پاسخ به این پرسش، یادگیری شیوه کار معماری Elm در پس‌زمینه کمک می‌کند. این کار، توضیح می‌دهد که چرا برخی چیزها کمی متفاوت از زبان‌هایی مانند جاوااسکریپت و پایتون کار می‌کنند.

## `sandbox`

درباره آن زیاد صحبت نکرده‌ام، اما تا به حال تمام برنامه‌های ما با [`Browser.sandbox`][browser.sandbox]{: .external } ایجاد شده‌اند. با ارایه یک `Model` اولیه، توضیح می‌دهیم که چگونه آن را `update` و `view` کنیم.

می‌توانید `Browser.sandbox` را مانند سیستم زیر تصور کنید:

![Browser.sandbox](../assets/diagrams/sandbox.svg)

با نوشتن توابع و تبدیل داده‌ها، در دنیای Elm باقی می‌مانیم. این فرآیند به **سیستم زمان اجرای** Elm متصل می‌شود. این سیستم مشخص می‌کند که چگونه `Html` را به طور کارآمد پردازش کند. آیا چیزی تغییر کرده است؟ حداقل تغییرات مورد نیاز DOM چقدر است؟ همچنین مشخص می‌کند که چه زمانی کسی روی یک دکمه کلیک یا در یک فیلد متنی تایپ می‌کند. این تعامل را به یک نوع داده `Msg` تبدیل کرده و به برنامه می‌فرستد.

با جداسازی دستکاری‌های DOM، امکان استفاده از بهینه‌سازی‌های قدرتمند فراهم می‌شود. بنابراین سیستم زمان اجرا بخش بزرگی از این دلیل این است که چرا Elm [یکی از سریع‌ترین گزینه‌های موجود به حساب می‌آید][benchmark]{: .external }.

## `element`

در نمونه‌های آتی، از [`Browser.element`][browser.element]{: .external } برای ایجاد برنامه استفاده خواهیم کرد. این کار، ایده‌های **دستور** و **اشتراک** را معرفی می‌کند که به ما اجازه می‌دهند با دنیای بیرون تعامل داشته باشیم.

می‌توانید `Browser.element` را مانند سیستم زیر تصور کنید:

![Browser.element](../assets/diagrams/element.svg)

علاوه بر تولید `Html`، برنامه همچنین مقادیر `Cmd` و `Sub` را به سیستم زمان اجرا می‌فرستد. در این فرآیند، برنامه می‌تواند به سیستم زمان اجرا **دستور** دهد تا یک درخواست HTTP انجام دهد یا یک مقدار تصادفی تولید کند. همچنین می‌تواند نسبت به فراخوانی زمان فعلی سیستم **اشتراک** ثبت کند.

فکر می‌کنم دستور و اشتراک زمانی معنا پیدا می‌کنند که کاربرد واقعی آن‌ها را مشاهده کنید. پس بیایید این کار را انجام دهیم!

!!! tip "نکته اول"

	برخی از خوانندگان ممکن است نگران اندازه برنامه باشند. "یک سیستم زمان اجرا؟ به نظر بزرگ می‌رسد!" اینطور نیست! در واقع، اندازه و حجم برنامه‌های Elm در مقایسه با گزینه‌های محبوب [به طرز استثنایی کمتر است][small-assets]{: .external }.

!!! tip "نکته دوم"

	در نمونه‌های آتی، از بسته‌های موجود در وبسایت [`package.elm-lang.org`][package]{: .external } استفاده خواهیم کرد. قبلا با چندتا کار کرده‌ایم:

	- [`elm/core`](https://package.elm-lang.org/packages/elm/core/latest/){: .external }
	- [`elm/html`](https://package.elm-lang.org/packages/elm/html/latest/){: .external }

	اما حالا شروع به استفاده از بسته‌های پیشرفته‌تری خواهیم کرد:

	- [`elm/http`](https://package.elm-lang.org/packages/elm/http/latest/){: .external }
	- [`elm/json`](https://package.elm-lang.org/packages/elm/json/latest/){: .external }
	- [`elm/random`](https://package.elm-lang.org/packages/elm/random/latest/){: .external }
	- [`elm/time`](https://package.elm-lang.org/packages/elm/time/latest/){: .external }

	با این حال، بسته‌های بیشتری در وبسایت `package.elm-lang.org` وجود دارند! بنابراین، وقتی برنامه Elm را به صورت محلی ایجاد می‌کنید، احتمالا شامل اجرای برخی دستورات در ترمینال خواهد بود:

	```bash
	elm init
	elm install elm/http
	elm install elm/random
	```

	این کار، یک فایل `elm.json` با وابستگی‌های مربوط به `elm/http` و `elm/random` برپا می‌کند.

	در نمونه‌های آتی، بسته‌هایی را که استفاده می‌کنیم ذکر خواهم کرد. امیدوارم این توضیحات کمی درباره این فرآیند زمینه‌سازی کرده باشد!

[browser.sandbox]: https://package.elm-lang.org/packages/elm/browser/latest/Browser#sandbox
[benchmark]: https://elm-lang.org/blog/blazing-fast-html-round-two
[browser.element]: https://package.elm-lang.org/packages/elm/browser/latest/Browser#element
[small-assets]: https://elm-lang.org/blog/small-assets-without-the-headache
[package]: https://package.elm-lang.org