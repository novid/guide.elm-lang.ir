# HTTP

اغلب اوقات مفید است که اطلاعاتی را از اینترنت دریافت کنیم.

برای نمونه، فرض کنید می‌خواهیم متن کامل کتاب _نظریه عمومی_ اثر والتر لیپمن را بارگیری کنیم. این کتاب که در سال ۱۹۲۲ منتشر شده است، دیدگاه تاریخی درباره ظهور رسانه‌های جمعی و پیامدهای آن برای دموکراسی ارایه می‌دهد. در ادامه، بر روی شیوه استفاده از بسته [`elm/http`][http] برای وارد کردن این کتاب به برنامه‌مان تمرکز خواهیم کرد!

برای مرور این برنامه در ویرایشگر آنلاین، روی دکمه آبی "ویرایش" کلیک کنید. احتمالا قبل از اینکه متن کامل کتاب نمایش داده شود، عبارت "در حال بارگیری..." نمایش می‌یابد. **اکنون روی دکمه آبی کلیک کنید!**

[http]: https://package.elm-lang.org/packages/elm/http/latest

<div class="edit-link"><a href="https://elm-lang.org/examples/book">ویرایش</a></div>

```elm
import Browser
import Html exposing (Html, text, pre)
import Http



-- MAIN


main =
  Browser.element
    { init = init
    , update = update
    , subscriptions = subscriptions
    , view = view
    }



-- MODEL


type Model
  = Failure
  | Loading
  | Success String


init : () -> (Model, Cmd Msg)
init _ =
  ( Loading
  , Http.get
      { url = "https://elm-lang.org/assets/public-opinion.txt"
      , expect = Http.expectString GotText
      }
  )



-- UPDATE


type Msg
  = GotText (Result Http.Error String)


update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    GotText result ->
      case result of
        Ok fullText ->
          (Success fullText, Cmd.none)

        Err _ ->
          (Failure, Cmd.none)



-- SUBSCRIPTIONS


subscriptions : Model -> Sub Msg
subscriptions model =
  Sub.none



-- VIEW


view : Model -> Html Msg
view model =
  case model of
    Failure ->
      text "I was unable to load your book."

    Loading ->
      text "Loading..."

    Success fullText ->
      pre [] [ text fullText ]
```

برخی از قسمت‌های برنامه، از مثال‌های قبلی معماری Elm، باید برای شما آشنا باشد. هنوز یک `Model` از برنامه‌، یک تابع `update` برای بروزرسانی پیام‌ها و یک تابع `view` برای بروزرسانی صفحه نمایش داریم.

قسمت‌های جدید، از جمله اضافه شدن تابع `subscription`، الگوی اصلی که قبلا دیدیم را با برخی تغییرات در `init` و `update` گسترش می‌دهند.

## `init`

تابع `init` چگونگی راه‌اندازی برنامه را توصیف می‌کند:

```elm
init : () -> (Model, Cmd Msg)
init _ =
  ( Loading
  , Http.get
      { url = "https://elm-lang.org/assets/public-opinion.txt"
      , expect = Http.expectString GotText
      }
  )
```

مثل همیشه، باید `Model` اولیه را تولید کنیم. همچنین اقدام به ایجاد یک **دستور**، از آنچه می‌خواهیم بلافاصله انجام شود، می‌کنیم. آن دستور در نهایت یک `Msg` تولید خواهد کرد که به تابع `update` داده می‌شود.

وبسایت کتاب در حالت `Loading` شروع می‌شود و می‌خواهیم متن کامل کتاب را GET کنیم. هنگام انجام یک درخواست GET با [`Http.get`][get]، ما `url` داده‌ای که می‌خواهیم بارگیری شود همراه با نوع آن داده را مشخص و `expect` می‌کنیم. بنابراین، `url` به برخی داده‌ها در وبسایت Elm اشاره دارد و **انتظار** داریم که یک `String` بزرگ باشد که می‌توانیم روی صفحه نمایش دهیم.

خط `Http.expectString GotText` کمی بیشتر از این می‌گوید که ما `expect` یک `String` داریم. همچنین می‌گوید در زمان دریافت پاسخ، باید به یک پیام `GotText` تبدیل شود:

```elm
type Msg
  = GotText (Result Http.Error String)

-- GotText (Ok "The Project Gutenberg EBook of ...")
-- GotText (Err Http.NetworkError)
-- GotText (Err (Http.BadStatus 404))
```

توجه داشته باشید که از نوع داده `Result` استفاده می‌کنیم. این کار، به ما اجازه می‌دهد تا تمام حالت‌های ممکن را در تابع `update` در نظر بگیریم. نوبت به تابع `update` رسیده است...

[get]: https://package.elm-lang.org/packages/elm/http/latest/Http#get

> **توجه:** اگر می‌پرسید چرا `init` یک تابع است (و چرا  آرگومانش را نادیده می‌گیریم)، در فصل آینده درباره تعامل با جاوا اسکریپت صحبت خواهیم کرد! ( آرگومان به ما اجازه می‌دهد تا اطلاعاتی را از جاوا اسکریپت در زمان راه‌اندازی دریافت کنیم.)

## `update`

تابع `update` نیز اطلاعات بیشتری را برمی‌گرداند:

```elm
update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    GotText result ->
      case result of
        Ok fullText ->
          (Success fullText, Cmd.none)

        Err _ ->
          (Failure, Cmd.none)
```

با نگاهی به نشانه‌گذاری نوع داده، می‌بینیم که فقط یک مدل بروز شده را برنمی‌گردانیم. _همچنین_ یک **دستور** از آنچه می‌خواهیم Elm انجام دهد تولید می‌کنیم.

به پیاده‌سازی که می‌پردازیم، به طور معمول، برای مرور پیام‌های مختلف از تکنیک تطبیق الگو استفاده می‌کنیم. وقتی یک پیام `GotText` دریافت می‌شود، `Result` یا نتیجه درخواست HTTP را بررسی و مدل را بسته به اینکه آیا موفقیت‌آمیز بوده یا خیر، بروزرسانی می‌کنیم. قسمت جدید، افزودن یک دستور به خروجی تابع `update` است.

در صورتی که متن کامل کتاب را با موفقیت دریافت کردیم، با فراخوانی دستور `Cmd.none` نشان می‌دهیم که کار دیگری برای انجام دادن وجود ندارد. متن کامل کتاب دریافت شده است!

در صورتی که خطایی وجود داشته باشد، با فراخوانی دستور `Cmd.none` نشان می‌دهیم که کار دیگری نمی‌توان انجام داد. متن کامل کتاب دریافت نشده است. اگر می‌خواستیم کمی پیشرفته‌تر عمل کنیم، می‌توانستیم بر روی [`Http.Error`][Error] از تکنیک تطبیق الگو استفاده کرده و در صورت بروز خطای `timeout`، دوباره درخواست بارگیری کتاب را صادر کنیم.

نکته این است که هر طور تصمیم بگیریم مدل خود را بروزرسانی کنیم، آزاد هستیم که دستورات جدیدی صادر کنیم. من به داده‌های بیشتری نیاز دارم! من یک عدد تصادفی می‌خواهم! و مواردی از این قبیل.

[Error]: https://package.elm-lang.org/packages/elm/http/latest/Http#Error

## `subscription`

مورد جدید دیگری که در این برنامه وجود دارد، تابع `subscription` است. این تابع به شما اجازه می‌دهد تا به `Model` نگاه کنید و تصمیم بگیرید آیا می‌خواهید به اطلاعات خاصی مشترک شوید یا خیر. در این برنامه، با استفاده از اشتراک `Sub.none` نشان می‌دهیم که نیازی به مشترک شدن در جایی نیست، اما به زودی یک مثال از برنامه ساعت خواهیم دید که می‌خواهیم به زمان فعلی مشترک شویم!

## خلاصه

در زمان ایجاد یک برنامه با `Browser.element`، یک سیستم به این شکل راه‌اندازی می‌شود:

![](element.svg)

در این حالت، توانایی صادر کردن **دستور** از تابع `init` و `update` را به دست می‌آوریم. این کار به ما اجازه می‌دهد تا هر زمان بخواهیم درخواست HTTP انجام دهیم. همچنین توانایی **مشترک شدن** به داده‌های جالب از سمت مرورگر وب را به دست می‌آوریم. (در ادامه یک مثال از این حالت خواهیم دید!)