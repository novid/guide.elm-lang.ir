# پرچم

برای ارسال مقدار سفارشی به Elm در زمان راه‌اندازی برنامه، از Flag یا پرچم استفاده می‌شود.

کاربرد رایج آن شامل ارسال کلید API، متغیر محیطی و داده کاربری است. اگر HTML را به‌صورت داینامیک یا پویا تولید می‌کنید، این ویژگی می‌تواند مفید واقع شود. همچنین، در بارگیری اطلاعات ذخیره شده یا کش شده از [این نمونه `localStorage`][localstorage] به ما کمک می‌کند.

## پرچم در HTML
کد HTML مشابه قبل است و فقط یک آرگومان `flags` به تابع `Elm.Main.init()` اضافه می‌شود:

```html
<html>
<head>
  <meta charset="UTF-8">
  <title>Main</title>
  <script src="main.js"></script>
</head>

<body>
  <div id="myapp"></div>
  <script>
  var app = Elm.Main.init({
    node: document.getElementById('myapp'),
    flags: Date.now()
  });
  </script>
</body>
</html>
```

در این نمونه، زمان فعلی را به میلی‌ثانیه ارسال می‌کنیم، اما هر مقدار جاوااسکریپت قابل تبدیل به JSON، می‌تواند به‌عنوان پرچم ارسال شود.

> **توجه:** این داده اضافی "پرچم یا flag" نامیده می‌شود زیرا شبیه پرچم‌های خط فرمان است. شما می‌توانید `elm make src/Main.elm` را با برخی پرچم‌ها مانند `--optimize` و `--output=main.js` فراخوانی کرده تا عملکرد اولیه آن را تغییر دهید.

## پرچم در Elm

برای مدیریت پرچم در Elm، باید تابع `init` را کمی تغییر دهید:

```elm
module Main exposing (..)

import Browser
import Html exposing (Html, text)


-- MAIN

main : Program Int Model Msg
main =
  Browser.element
    { init = init
    , view = view
    , update = update
    , subscriptions = subscriptions
    }


-- MODEL

type alias Model = { currentTime : Int }

init : Int -> ( Model, Cmd Msg )
init currentTime =
  ( { currentTime = currentTime }
  , Cmd.none
  )


-- UPDATE

type Msg = NoOp

update : Msg -> Model -> ( Model, Cmd Msg )
update _ model =
  ( model, Cmd.none )


-- VIEW

view : Model -> Html Msg
view model =
  text (String.fromInt model.currentTime)


-- SUBSCRIPTIONS

subscriptions : Model -> Sub Msg
subscriptions _ =
  Sub.none
```

تنها نکته مهم این است که تابع `init` یک آرگومان `Int` می‌گیرد. به همین دلیل، کد Elm بلافاصله به پرچم‌هایی که از جاوااسکریپت ارسال می‌کنید، دسترسی پیدا می‌کند. در ادامه، می‌توانید مدل خود را سفارشی یا برخی دستورات را اجرا کنید، هر چیزی که برنامه به آن نیاز داشته باشد.

توصیه می‌کنم [این نمونه `localStorage`][localStorage] را برای کاربرد جالب‌تری از پرچم‌ها بررسی کنید!

## تایید پرچم

اما چه اتفاقی می‌افتد اگر `init` یک پرچم با نوع داده `Int` بگیرد، اما فراخوانی آن به صورت `Elm.Main.init({ flags: "haha, what now?" })` باشد؟

Elm با بررسی نوع داده ورودی، اطمینان حاصل می‌کند که پرچم‌ها دقیقا همان چیزی هستند که انتظار دارید. بدون این بررسی، می‌توانید هر چیزی را ارسال کنید که منجر به بروز خطای زمان اجرا در Elm می‌شود!

انواع داده‌‌ مختلفی وجود دارند که می‌توان به‌عنوان پرچم از آن‌ها استفاده کرد:

- `Bool`
- `Int`
- `Float`
- `String`
- `Maybe`
- `List`
- `Array`
- `Tuple`
- `Record`
- [`Json.Decode.Value`][json.decode.value]

بسیاری از توسعه‌دهندگان همیشه از `Json.Decode.Value` استفاده می‌کنند زیرا کنترل دقیقی به آن‌ها می‌دهد. آن‌ها می‌توانند یک دیکودر بنویسند تا هر سناریوی عجیبی را در Elm مدیریت کرده و برای بازیابی داده‌های غیرمنتظره به خوبی آماده باشند.

سایر انواع داده، در واقع قبل از اینکه راهی برای دیکودرهای JSON پیدا کنیم، آمده‌اند. اگر تصمیم به استفاده از آن‌ها گرفتید، برخی نکات ظریف وجود دارد که باید به آن‌ها توجه کنید. در هر یک از نمونه‌های زیر، نوع داده پرچم مورد نظر و در زیر آن مقادیر مختلف جاوااسکریپت محاسبه می‌شوند:

- `init : Int -> ...`
  - `0` => `0`
  - `7` => `7`
  - `3.14` => خطا
  - `6.12` => خطا

- `init : Maybe Int -> ...`
  - `null` => `Nothing`
  - `42` => `Just 42`
  - `"hi"` => خطا

- `init : { x : Float, y : Float } -> ...`
  - `{ x: 3, y: 4, z: 50 }` => `{ x = 3, y = 4 }`
  - `{ x: 3, name: "Tom" }` => خطا
  - `{ x: 360, y: "why?" }` => خطا

- `init : (String, Int) -> ...`
  - `["Tom", 42]` => `("Tom", 42)`
  - `["Sue", 33]` => `("Sue", 33)`
  - `["Bob", "4"]` => خطا
  - `["Joe", 9, 9]` => خطا

توجه داشته باشید که وقتی یکی از تبدیل‌ها اشتباه می‌شود، **یک خطا در سمت جاوااسکریپت دریافت می‌کنید!** در اینجا، سیاست "fail fast یا شکست سریع" را اتخاذ کرده‌ایم. به محض بروز خطا، گزارش آن ارسال می‌شود، بجای اینکه خطا به تدریج در کد Elm پیشروی کند. این یکی دیگر از دلایلی است که توسعه‌دهندگان دوست دارند از `Json.Decode.Value` برای پرچم‌ها استفاده کنند. بجای اینکه در جاوااسکریپت خطا بگیرید، مقدار غیر منتظره از طریق یک دیکودر عبور کرده و تضمین می‌کند که عملکردی جایگزین برای آن پیاده‌سازی شده باشد.

[localstorage]: https://github.com/elm-community/js-integration-examples/tree/master/localStorage
[json.decode.value]: https://package.elm-lang.org/packages/elm/json/latest/Json-Decode#Value