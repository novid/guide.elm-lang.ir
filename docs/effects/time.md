# Time

اکنون می‌خواهیم یک ساعت دیجیتال بسازیم. (ساعت آنالوگ یک تمرین خواهد بود!)

تا به حال روی دستورات تمرکز داشتیم. با مثال‌های HTTP و عدد تصادفی، به Elm دستور دادیم که کار خاصی را بلافاصله انجام دهد، اما این الگوی عجیبی برای یک ساعت است. ما _همیشه_ می‌خواهیم زمان فعلی را بدانیم. اینجاست که مفهوم **اشتراک یا subscription** وارد می‌شود!

با کلیک روی دکمه آبی "ویرایش" شروع و کد برنامه را مطالعه کنید.

<div class="edit-link"><a href="https://elm-lang.org/examples/time">ویرایش</a></div>

```elm
import Browser
import Html exposing (..)
import Task
import Time



-- MAIN


main =
  Browser.element
    { init = init
    , view = view
    , update = update
    , subscriptions = subscriptions
    }



-- MODEL


type alias Model =
  { zone : Time.Zone
  , time : Time.Posix
  }


init : () -> (Model, Cmd Msg)
init _ =
  ( Model Time.utc (Time.millisToPosix 0)
  , Task.perform AdjustTimeZone Time.here
  )



-- UPDATE


type Msg
  = Tick Time.Posix
  | AdjustTimeZone Time.Zone



update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    Tick newTime ->
      ( { model | time = newTime }
      , Cmd.none
      )

    AdjustTimeZone newZone ->
      ( { model | zone = newZone }
      , Cmd.none
      )



-- SUBSCRIPTIONS


subscriptions : Model -> Sub Msg
subscriptions model =
  Time.every 1000 Tick



-- VIEW


view : Model -> Html Msg
view model =
  let
    hour   = String.fromInt (Time.toHour   model.zone model.time)
    minute = String.fromInt (Time.toMinute model.zone model.time)
    second = String.fromInt (Time.toSecond model.zone model.time)
  in
  h1 [] [ text (hour ++ ":" ++ minute ++ ":" ++ second) ]
```

مفاهیم جدید همه از بسته [`elm/time`][time] می‌آیند. بیایید این بخش‌ها را بررسی کنیم!

[time]: https://package.elm-lang.org/packages/elm/time/latest/

## `Time.Posix` و `Time.Zone`

برای کار با زمان در برنامه‌نویسی، به سه مفهوم مختلف نیاز داریم:

- **زمان انسانی** &mdash; این همان چیزی است که بر روی ساعت (۸ صبح) یا تقویم (۳ می) می‌بینید. عالی! اما اگر تماس من در ساعت ۸ صبح در بوستون باشد، برای دوستم در ونکوور چه ساعتی است؟ اگر در ساعت ۸ صبح در توکیو باشد، آیا این همان روز در نیویورک است؟ (نه!) بنابراین بین [منطقه‌های زمانی][tz] بر اساس مرزهای سیاسی در حال تغییر و استفاده نامنظم از [ساعت تابستانی][dst]، زمان انسانی هرگز نباید در `Model` یا پایگاه داده ذخیره شود! این زمان، فقط برای نمایش است!

- **زمان POSIX** &mdash; با زمان POSIX، مهم نیست کجا زندگی می‌کنید یا چه زمانی از سال است. این فقط تعداد ثانیه‌هایی است که از یک لحظه دلخواه (در سال ۱۹۷۰) گذشته است. در هر جایی که بروید، زمان POSIX یکسان است.

- **منطقه زمانی** &mdash; یک "منطقه زمانی" مجموعه‌ای از داده‌ها است که به شما اجازه می‌دهد زمان POSIX را به زمان انسانی تبدیل کنید. این _فقط_ `UTC-7` یا `UTC+3` نیست! منطقه‌های زمانی بسیار پیچیده‌تر از یک جابجایی ساده هستند! هر بار که [فلوریدا به طور دائمی به DST تغییر می‌کند][florida] یا [ساموآ از UTC-11 به UTC+13 تغییر می‌کند][samoa]، یک انسان بیچاره یادداشتی به [پایگاه داده منطقه زمانی IANA][iana] اضافه می‌کند. آن پایگاه داده بر روی هر کامپیوتری بارگیری می‌شود و بین زمان POSIX و تمام موارد خاص در پایگاه داده، می‌توانیم زمان انسانی را محاسبه کنیم!

بنابراین، برای نشان دادن زمان به انسان، باید همیشه `Time.Posix` و `Time.Zone` را بدانید. همین! تمام مطالبی که درباره "زمان انسانی" بیان شد، برای تابع `view` است، نه`Model`. در واقع، می‌توانید این را در تابع `view` ببینید:

```elm
view : Model -> Html Msg
view model =
  let
    hour   = String.fromInt (Time.toHour   model.zone model.time)
    minute = String.fromInt (Time.toMinute model.zone model.time)
    second = String.fromInt (Time.toSecond model.zone model.time)
  in
  h1 [] [ text (hour ++ ":" ++ minute ++ ":" ++ second) ]
```

تابع [`Time.toHour`][toHour] آرگومان‌های `Time.Zone` و `Time.Posix` را می‌گیرد و یک `Int` از `0` تا `23` به ما برمی‌گرداند که نشان می‌دهد در _منطقه زمانی_ شما چه ساعتی است.

اطلاعات بیشتری درباره مدیریت زمان در فایل README بسته [`elm/time`][time] وجود دارد. حتما قبل از انجام کارهای بیشتر با زمان، آن را بخوانید! به ویژه اگر با زمان‌بندی، تقویم و موارد مشابه کار می‌کنید.

[tz]: https://en.wikipedia.org/wiki/Time_zone  
[dst]: https://en.wikipedia.org/wiki/Daylight_saving_time  
[iana]: https://en.wikipedia.org/wiki/IANA_time_zone_database  
[samoa]: https://en.wikipedia.org/wiki/Time_in_Samoa  
[florida]: https://www.npr.org/sections/thetwo-way/2018/03/08/591925587/  
[toHour]: https://package.elm-lang.org/packages/elm/time/latest/Time#toHour

## `اشتراک یا subscription`

خوب، حالا چگونه باید `Time.Posix` را دریافت کنیم؟ با یک **subscription**!

```elm
subscriptions : Model -> Sub Msg
subscriptions model =
  Time.every 1000 Tick
```

ما از تابع [`Time.every`][every] استفاده می‌کنیم:

[every]: https://package.elm-lang.org/packages/elm/time/latest/Time#every

```elm
every : Float -> (Time.Posix -> msg) -> Sub msg
```

این تابع دو آرگومان می‌گیرد:

1. یک بازه زمانی به میلی‌ثانیه. `1000` میلی‌ثانیه که به معنای هر ثانیه است. اما می‌توانیم بگوییم `60 * 1000` برای هر دقیقه یا `5 * 60 * 1000` برای هر پنج دقیقه.
2. یک تابع که زمان فعلی را به یک `Msg` تبدیل می‌کند. هر ثانیه، زمان فعلی به یک `Tick <time>` برای تابع `update` تبدیل می‌شود.

این الگوی پایه، برای هر اشتراکی بکار می‌رود. با پیکربندی یک مقدار، توصیف می‌کنید چگونه مقدار `Msg` تولید شود. خیلی هم بد نیست!

## `Task.perform`

دریافت `Time.Zone` کمی پیچیده‌تر است. برنامه یک **دستور** به این شکل ایجاد کرد:

```elm
Task.perform AdjustTimeZone Time.here
```

مطالعه مستندات ماژول [`Task`][task] بهترین راه برای درک این خط است. مستندات، به طور خاص، برای توضیح مفاهیم جدید نوشته شده‌اند و فکر می‌کنم بیان مجدد آن مفاهیم در این قسمت، خارج از موضوع این قسمت باشد. نکته این است که ما به سیستم زمان اجرا دستور می‌دهیم تا `Time.Zone` را هر جا که کد در حال اجرا است، به ما بدهد.

[utc]: https://package.elm-lang.org/packages/elm/time/latest/Time#utc  
[task]: https://package.elm-lang.org/packages/elm/core/latest/Task

> **تمرین:**
>
> - یک دکمه اضافه کنید تا ساعت را متوقف و اشتراک `Time.every` را خاموش کند.
> - با استفاده از ویژگی‌های [`style`][style]، ساعت دیجیتال را زیباتر کنید.
> - از بسته [`elm/svg`][svg] استفاده کنید تا یک ساعت آنالوگ با عقربه ثانیه شمار قرمز بسازید!

[style]: https://package.elm-lang.org/packages/elm/html/latest/Html-Attributes#style  
[svg]: https://package.elm-lang.org/packages/elm/svg/latest/