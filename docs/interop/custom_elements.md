# Custom Elements

در قسمت‌های قبل (۱) شیوه راه‌اندازی برنامه، (۲) شیوه ارسال داده به عنوان پرچم در زمان راه‌اندازی و (۳) شیوه ارسال پیام بین Elm و JS را با استفاده از پورت مشاهده کردیم. درست حدس زدید! یک راه دیگر برای تعامل بین Elm و JS وجود دارد.

به نظر می‌رسد مرورگرها به طور فزاینده‌ای از [عناصر سفارشی](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements) پشتیبانی می‌کنند. اینکار برای گنجاندن جاوا اسکریپت در برنامه‌های Elm بسیار مفید است.

در این قسمت، یک [نمونه کوچک](https://github.com/elm-community/js-integration-examples/tree/master/internationalization) استفاده از عناصر سفارشی برای انجام برخی عملیات بومی‌سازی (l10n) و بین‌المللی‌سازی (i18n) آورده شده است.

## ایجاد عناصر سفارشی

فرض کنید می‌خواهیم تاریخ را بومی‌سازی کنیم، اما هنوز عملکرد آن به زبان Elm پیاده‌سازی نشده است. شاید بخواهید تابعی بنویسید که تاریخ را بومی‌سازی کند:

```javascript
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

اما چگونه می‌توانیم از این تابع در Elm استفاده کنیم؟! مرورگرهای جدید به شما اجازه می‌دهند انواع جدیدی از DOM Node (عنصر سفارشی) را به این شکل ایجاد کنید:

```javascript
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

اینکار را قبل از راه‌اندازی کد Elm انجام دهید تا بتوانید کدی مانند این را در Elm بنویسید:

```elm
import Html exposing (Html, node)
import Html.Attributes (attribute)

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

می‌توانید نسخه کامل این برنامه را [ از اینجا](https://github.com/elm-community/js-integration-examples/tree/master/internationalization) مشاهده کنید.

## اطلاعات بیشتر

Luke تجربه بیشتری در زمینه عناصر سفارشی دارد و فکر می‌کنم سخنرانی او در کنفرانس Elm Europe مقدمه‌ای عالی باشد!

{% youtube %} https://www.youtube.com/watch?v=tyFe9Pw6TVE {% endyoutube %}

مستندات مربوط به عناصر سفارشی ممکن است کمی گیج‌کننده باشد. اگر استفاده از عناصر سفارشی انتخاب مناسبی برای پروژه شما است، امیدوارم این مقدمه برای شروع گنجاندن منطقی ساده برای `Intl` یا حتی ویجت‌های بزرگ React کافی باشد.