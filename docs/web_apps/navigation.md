# پیمایش

به تازگی دیدیم که چگونه یک صفحه را میزبانی کنیم، اما فرض کنید که در حال ساخت یک وبسایت مانند `package.elm-lang.org` هستیم. این وبسایت دارای چندین صفحه است (برای نمونه [جستجو](https://package.elm-lang.org/)، [راهنما](https://package.elm-lang.org/packages/elm/core/latest/)، [مستندات](https://package.elm-lang.org/packages/elm/core/latest/Maybe)) که همه به طور متفاوتی کار می‌کنند. چگونه این کار انجام می‌شود؟

## ساختار چند صفحه‌ای

ساده‌ترین روش این است که تعدادی فایل HTML مختلف را میزبانی کنیم. به صفحه اصلی می‌روید؟ HTML جدید بارگیری کنید. به صفحه مستندات `elm/core` می‌روید؟ HTML جدید بارگیری کنید. به صفحه مستندات `elm/json` می‌روید؟ HTML جدید بارگیری کنید.

تا نسخه Elm 0.19، دقیقا همین کار انجام می‌شد! این کار ساده است، اما چند نقطه ضعف دارد:

1. **صفحه خالی:** با هر بار بارگیری HTML جدید، صفحه سفید می‌شود. آیا می‌توان بجای آن یک انتقال زیبا انجام داد؟
2. **درخواست اضافی:** هر بسته نرم‌افزاری یک فایل `docs.json` دارد، اما هر بار که به یک ماژول مانند [`String`](https://package.elm-lang.org/packages/elm/core/latest/String) یا [`Maybe`](https://package.elm-lang.org/packages/elm/core/latest/Maybe) مراجعه می‌کنید، بارگیری می‌شود. آیا می‌توان داده‌ها را بین صفحات به اشتراک گذاشت؟
3. **کد اضافی:** صفحه اصلی و مستندات، توابع مشترکی مانند `Html.text` و `Html.div` دارند. آیا می‌توان این کد را بین صفحات به اشتراک گذاشت؟

می‌توانیم هر سه مورد را بهبود ببخشیم! ایده اصلی این است که HTML را فقط یک مرتبه بارگیری، سپس کمی با URL تغییرات را مدیریت کنیم.

## ساختار تک صفحه‌ای

بجای ایجاد برنامه با `Browser.element` یا `Browser.document`، می‌توانیم یک [`Browser.application`](https://package.elm-lang.org/packages/elm/browser/latest/Browser#application) ایجاد کرده تا از بارگیری HTML جدید هنگام تغییر URL جلوگیری کنیم:

```elm
application :
  { init : flags -> Url -> Key -> ( model, Cmd msg )
  , view : model -> Document msg
  , update : msg -> model -> ( model, Cmd msg )
  , subscriptions : model -> Sub msg
  , onUrlRequest : UrlRequest -> msg
  , onUrlChange : Url -> msg
  }
  -> Program flags model msg
```

این کار منجر به گسترش عملکرد `Browser.document` در سه سناریوی مهم می‌شود:

**زمانی که برنامه شروع می‌شود**، تابع `init` مقدار URL فعلی را از نوار پیمایش مرورگر دریافت می‌کند. این کار به شما اجازه می‌دهد با توجه به مقدار `Url` چیزهای مختلفی را نمایش دهید.

**زمانی که روی یک لینک کلیک می‌شود**، مانند `<a href="/home">Home</a>`، یک درخواست [`UrlRequest`][ur] صادر می‌شود. بنابراین، بجای بارگیری HTML جدید با تمام معایب آن، `onUrlRequest` یک پیام برای تابع `update` ایجاد می‌کند که می‌توان دقیقا تصمیم گرفت چه کاری باید انجام شود. برای نمونه، می‌توان موقعیت اسکرول را ذخیره‌سازی کرد، داده‌ها را در مرورگر حفظ کرد یا URL را تغییر داد.

**زمانی که URL تغییر می‌کند**، `Url` جدید به `onUrlChange` ارسال می‌شود. پیام حاصل به تابع `update` می‌رود که در آن می‌توان تصمیم گرفت چگونه صفحه جدید را نمایش دهیم.

بنابراین، بجای بارگیری HTML جدید، این سه قابلیت جدید به شما کنترل کامل بر تغییرات URL را می‌دهند. بیایید آن را در عمل ببینیم!

[u]: https://package.elm-lang.org/packages/elm/url/latest/Url#Url
[ur]: https://package.elm-lang.org/packages/elm/browser/latest/Browser#UrlRequest
[bn]: https://package.elm-lang.org/packages/elm/browser/latest/Browser-Navigation
[bnp]: https://package.elm-lang.org/packages/elm/browser/latest/Browser-Navigation#pushUrl
[bnl]: https://package.elm-lang.org/packages/elm/browser/latest/Browser-Navigation#load

## نمونه کد

با برنامه پایه `Browser.application` شروع می‌کنیم. این برنامه فقط URL فعلی را پیگیری می‌کند. اکنون کد را مرور کنید! تقریبا تمام چیزهای جدید و جالب در تابع `update` اتفاق می‌افتد که در ادامه به جزییات آن خواهیم پرداخت:

```elm
import Browser
import Browser.Navigation as Nav
import Html exposing (..)
import Html.Attributes exposing (..)
import Url

-- MAIN

main : Program () Model Msg
main =
  Browser.application
    { init = init
    , view = view
    , update = update
    , subscriptions = subscriptions
    , onUrlChange = UrlChanged
    , onUrlRequest = LinkClicked
    }

-- MODEL

type alias Model =
  { key : Nav.Key
  , url : Url.Url
  }

init : () -> Url.Url -> Nav.Key -> ( Model, Cmd Msg )
init flags url key =
  ( Model key url, Cmd.none )

-- UPDATE

type Msg
  = LinkClicked Browser.UrlRequest
  | UrlChanged Url.Url

update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
  case msg of
    LinkClicked urlRequest ->
      case urlRequest of
        Browser.Internal url ->
          ( model, Nav.pushUrl model.key (Url.toString url) )

        Browser.External href ->
          ( model, Nav.load href )

    UrlChanged url ->
      ( { model | url = url }
      , Cmd.none
      )

-- SUBSCRIPTIONS

subscriptions : Model -> Sub Msg
subscriptions _ =
  Sub.none

-- VIEW

view : Model -> Browser.Document Msg
view model =
  { title = "URL Interceptor"
  , body =
      [ text "The current URL is: "
      , b [] [ text (Url.toString model.url) ]
      , ul []
          [ viewLink "/home"
          , viewLink "/profile"
          , viewLink "/reviews/the-century-of-the-self"
          , viewLink "/reviews/public-opinion"
          , viewLink "/reviews/shah-of-shahs"
          ]
      ]
  }

viewLink : String -> Html msg
viewLink path =
  li [] [ a [ href path ] [ text path ] ]
```

تابع `update` می‌تواند پیام‌های `LinkClicked` یا `UrlChanged` را مدیریت کند. در شاخه `LinkClicked` چیزهای جدید و جالب زیادی وجود دارد، بنابراین ابتدا بر روی آن تمرکز می‌کنیم!

## `UrlRequest`

هر زمان که روی یک لینک مانند `<a href="/home">/home</a>` کلیک شود، یک مقدار `UrlRequest` تولید می‌شود:

```elm
type UrlRequest
  = Internal Url.Url
  | External String
```

نوع `Internal` برای هر لینکی است که در همان دامنه باقی می‌ماند. بنابراین، اگر در حال مرور `https://example.com` هستید، لینک‌های داخلی شامل چیزهایی مانند `settings#privacy`، `/home`، `https://example.com/home` و `//example.com/home` هستند.

نوع `External` برای هر لینکی است که به دامنه‌ای دیگر می‌رود. لینک‌هایی مانند `https://elm-lang.org/examples`، `https://static.example.com` و `http://example.com/home` همه به دامنه‌ای دیگر می‌روند. توجه داشته باشید که تغییر پروتکل از `https` به `http` به عنوان یک دامنه متفاوت در نظر گرفته می‌شود!

هر لینکی که کلیک شود، برنامه یک پیام `LinkClicked` ایجاد کرده و آن را به تابع `update` ارسال می‌کند. اینجاست که بیشتر کدهای جدید و جالب را می‌بینیم!

### `LinkClicked`

بیشتر منطق تابع `update` تصمیم‌گیری درباره این است که با مقادیر `UrlRequest` چه کاری انجام دهیم:

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
  case msg of
    LinkClicked urlRequest ->
      case urlRequest of
        Browser.Internal url ->
          ( model, Nav.pushUrl model.key (Url.toString url) )

        Browser.External href ->
          ( model, Nav.load href )

    UrlChanged url ->
      ( { model | url = url }
      , Cmd.none
      )
```

دو تابع `Nav.pushUrl` و `Nav.load` عملکرد جالبی دارند. این‌ها هر دو از ماژول [`Browser.Navigation`](https://package.elm-lang.org/packages/elm/browser/latest/Browser-Navigation) هستند که درباره تغییر URL به روش‌های مختلف است. ما از دو تابع رایج آن ماژول استفاده می‌کنیم:

- تابع [`pushUrl`](https://package.elm-lang.org/packages/elm/browser/latest/Browser-Navigation#pushUrl) مقدار URL را تغییر می‌دهد، اما HTML جدید بارگیری نمی‌شود. در عوض، یک پیام `UrlChanged` را ایجاد می‌کند که خودمان آن را مدیریت می‌کنیم! همچنین یک ورودی به "تاریخچه مرورگر" اضافه می‌کند تا وقتی که کاربران دکمه‌های `BACK` یا `FORWARD` را فشار می‌دهند، همه چیز به طور عادی کار کند.
- تابع [`load`](https://package.elm-lang.org/packages/elm/browser/latest/Browser-Navigation#load) تمام HTML جدید را بارگیری می‌کند. این کار معادل تایپ کردن URL در نوار پیمایش و فشردن Enter است. بنابراین، هر چیزی که در `Model` حاضر وجود دارد، دور ریخته و یک صفحه کاملا جدید بارگیری می‌شود.

بنابراین با نگاهی به تابع `update`، اکنون می‌توانیم درک بهتری از چگونگی ارتباط اجزا با یکدیگر داشته باشیم. وقتی کاربر روی لینک `/home` کلیک می‌کند، یک پیام `Internal` دریافت و از `pushUrl` برای تغییر URL _بدون_ بارگیری HTML جدید استفاده می‌کنیم. وقتی کاربر روی لینک `https://elm-lang.org` کلیک می‌کند، یک پیام `External` دریافت و از `load` برای بارگیری HTML جدید در سرور استفاده می‌کنیم.

> **نکته ۱:** هر دو لینک `Internal` و `External` بلافاصله دستورات خود را تولید می‌کنند، اما این کار الزامی نیست! وقتی روی یک لینک `Internal` کلیک می‌شود، شاید بخواهید از [`getViewport`](https://package.elm-lang.org/packages/elm/browser/latest/Browser-Dom#getViewport) برای ذخیره‌سازی موقعیت اسکرول استفاده کنید تا در صورت فشردن دکمه `BACK`، آن را حفظ کنید. وقتی روی یک لینک `External` کلیک می‌شود، شاید بخواهید محتوای جعبه متن را قبل از رفتن به صفحه دیگر در پایگاه داده ذخیره‌سازی کنید. همه این‌ها ممکن است! این یک تابع `update` عادی است که می‌توانید وضعیت پیمایش را به تاخیر بیندازید و هر کاری که می‌خواهید انجام دهید.
>
> **نکته ۲:** اگر می‌خواهید "آنچه را که کاربران در حال مشاهده بودند" هنگام بازگشت به عقب بازیابی کنید، موقعیت اسکرول ایده‌آل نیست. اگر آن‌ها مرورگر خود را تغییر اندازه دهند یا دستگاه خود را بچرخانند، ممکن است این مقدار به اشتباه محاسبه شود! بنابراین، بهتر است "آنچه را که آن‌ها در حال مشاهده بودند" ذخیره‌سازی کنید. شاید این به معنای استفاده از تابع [`getViewportOf`](https://package.elm-lang.org/packages/elm/browser/latest/Browser-Dom#getViewportOf) باشد تا دقیقا بفهمید در حال حاضر چه چیزی روی صفحه نمایش قرار دارد. جزییات بستگی به این دارد که برنامه شما دقیقا چگونه کار می‌کند، بنابراین نمی‌توانم مشاوره دقیقی در این مورد بدهم!

## `UrlChanged`

چندین راه برای دریافت پیام‌های `UrlChanged` وجود دارد. به تازگی دیدیم که `pushUrl` آن‌ها را تولید می‌کند، اما فشردن دکمه‌های `BACK` و `FORWARD` مرورگر نیز آن‌ها را تولید می‌کند.همانطور که در نکات قبلی گفتم، وقتی یک پیام `LinkClicked` دریافت می‌کنید، ممکن است دستور `pushUrl` بلافاصله صادر نشود.

بنابراین نکته خوب در مورد داشتن یک پیام جداگانه `UrlChanged` این است که مهم نیست URL چگونه یا چه زمانی تغییر کرده است. تنها چیزی که باید بدانید این است که تغییر کرده است!

در نمونه کد این صفحه فقط URL جدید را ذخیره‌سازی می‌کنیم، اما در یک وب اپلیکیشن واقعی، شما باید URL را تجزیه و تحلیل کنید تا بفهمید چه محتوایی را نمایش دهید. در ادامه فصل، به این موضوع می‌پردازیم!

...
> **نکته:** در مورد [`Nav.Key`](https://package.elm-lang.org/packages/elm/browser/latest/Browser-Navigation#Key) صحبت نکردم تا بر روی مفاهیم مهم‌تر تمرکز کنم. اما برای کسانی که علاقه‌مند هستند، اینجا توضیح می‌دهم!
>
> یک `Key` یا کلید پیمایش برای ایجاد دستوراتی مانند `pushUrl`، که URL را تغییر می‌دهند، لازم است. فقط زمانی به یک `Key` دسترسی دارید که برنامه را با `Browser.application` ایجاد کنید. این کار تضمین می‌کند برنامه شما برای شناسایی تغییرات URL مجهز است. اگر مقادیر `Key` در انواع دیگر برنامه در دسترس بودند، برنامه‌نویسان بی‌خبر قطعا با برخی [باگ‌های آزاردهنده][bugs] مواجه می‌شدند و بسیاری از تکنیک‌ها را به سختی یاد می‌گرفتند!
>
> به همین دلیل، یک خط در تعریف `Model` برای `Key` داریم. این یک هزینه نسبتا کم برای کمک به جلوگیری از یک دسته مشکلات بسیار ظریف است!

[bugs]: https://github.com/elm/browser/blob/1.0.0/notes/navigation-in-elements.md