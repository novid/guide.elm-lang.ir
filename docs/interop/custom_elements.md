# عنصر سفارشی

در قسمت‌های قبل (۱) شیوه راه‌اندازی برنامه، (۲) شیوه ارسال داده به عنوان پرچم در زمان راه‌اندازی و (۳) شیوه ارسال پیام بین Elm و جاوااسکریپت را با استفاده از پورت مشاهده کردیم. درست حدس زدید، یک راه دیگر برای تعامل با جاوااسکریپت وجود دارد!

به نظر می‌رسد مرورگرهای وب به طور فزاینده‌ای از [Custom Elements][custom-elements] پشتیبانی می‌کنند. این کار برای گنجاندن جاوااسکریپت در برنامه‌های Elm بسیار مفید است.

در این قسمت، یک [نمونه کوچک][i18n] استفاده از عنصر سفارشی برای انجام برخی عملیات بومی‌سازی (l10n) و بین‌المللی‌سازی (i18n) آورده شده است.

## ایجاد عنصر سفارشی {#creating-custom-elements}

فرض کنید می‌خواهیم تاریخ را بومی‌سازی کنیم، اما هنوز عملکرد آن در Elm پیاده‌سازی نشده است. شاید بخواهید تابعی بنویسید که تاریخ را بومی‌سازی کند:

```javascript linenums="1"
//
//   localizeDate('sr-RS', 12, 5) === "петак, 1. јун 2012."
//   localizeDate('en-GB', 12, 5) === "Friday, 1 June 2012"
//   localizeDate('en-US', 12, 5) === "Friday, June 1, 2012"
//
function localizeDate(lang, year, month)
{
	const dateTimeFormat = new Intl.DateTimeFormat(lang, {
		weekday: 'long',
		year: 'numeric',
		month: 'long',
		day: 'numeric'
	});

	return dateTimeFormat.format(new Date(year, month));
}
```

اما چگونه می‌توانیم از این تابع در Elm استفاده کنیم؟! مرورگرهای وب به شما اجازه می‌دهند نوع جدیدی از DOM Node به نام عنصر سفارشی را به این شکل ایجاد کنید:

```javascript linenums="1"
//
//   <intl-date lang="sr-RS" year="2012" month="5">
//   <intl-date lang="en-GB" year="2012" month="5">
//   <intl-date lang="en-US" year="2012" month="5">
//
customElements.define('intl-date',
	class extends HTMLElement {
       // things required by Custom Elements
		constructor() { super(); }
		connectedCallback() { this.setTextContent(); }
		attributeChangedCallback() { this.setTextContent(); }
		static get observedAttributes() { return ['lang','year','month']; }

       // Our function to set the textContent based on attributes.
		setTextContent()
		{
			const lang = this.getAttribute('lang');
			const year = this.getAttribute('year');
			const month = this.getAttribute('month');
			this.textContent = localizeDate(lang, year, month);
		}
	}
);
```

توابع مهم در اینجا `attributeChangedCallback` و `observedAttributes` هستند. برای شناسایی تغییرات در ویژگی‌هایی که برای شما مهم هستند، به چنین عملکردی نیاز دارید.

این کار را قبل از راه‌اندازی کد Elm انجام دهید تا بتوانید کدی مانند این را در Elm بنویسید:

```elm linenums="1"
module Main exposing (..)

import Html exposing (Html, node)
import Html.Attributes exposing (attribute)


viewDate : String -> Int -> Int -> Html msg
viewDate lang year month =
    node "intl-date"
        [ attribute "lang" lang
        , attribute "year" (String.fromInt year)
        , attribute "month" (String.fromInt month)
        ]
        []
```

اکنون می‌توانید با فراخوانی تابع `viewDate`، به اطلاعات بومی‌سازی شده در قسمت `view` دسترسی داشته باشید.

می‌توانید نسخه کامل این برنامه را [از اینجا][i18n] مشاهده کنید.

## اطلاعات بیشتر {#more-info}

Luke Westby تجربه بیشتری در زمینه عنصر سفارشی دارد و فکر می‌کنم سخنرانی او در کنفرانس Elm Europe مقدمه‌ای عالی باشد!

مستندات مربوط به عنصر سفارشی ممکن است کمی گیج‌کننده باشد. اگر استفاده از آن انتخاب مناسبی برای پروژه شما است، امیدوارم این مقدمه برای شروع گنجاندن منطقی ساده برای ماژول `Intl` در مرورگر وب یا حتی ویجت‌های بزرگ React کافی باشد.
<iframe width="640" height="360" src="https://www.youtube-nocookie.com/embed/tyFe9Pw6TVE?si=S-N2nz-LEaRdEfM-" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

[custom-elements]: https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements
[i18n]: https://github.com/elm-community/js-integration-examples/tree/master/internationalization