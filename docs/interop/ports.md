# Ports

برای برقراری ارتباط بین Elm و JavaScript از Port یا پورت استفاده می‌شود.

پورت برای کاربردهای [`localStorage`](https://github.com/elm-community/js-integration-examples/tree/master/localStorage) و [`WebSockets`](https://github.com/elm-community/js-integration-examples/tree/master/websockets) استفاده می‌شود. بیایید بر روی مثال `WebSockets` تمرکز کنیم.

## کاربرد در JavaScript

در این مثال، تقریبا همان ساختار HTML را داریم که در مثال‌های قبلی استفاده کرده‌ایم، اما با کمی کد اضافی جاوا اسکریپت. با ایجاد یک اتصال به `wss://echo.websocket.org` هر چیزی را که به آن ارسال کنید، تکرار می‌کند. در [نمونه کد Ellie](https://ellie-app.com/8yYgw7y7sM2a1) می‌توانید ببینید این کار به ما اجازه می‌دهد تا اسکلت یک اتاق چت را بسازیم:

```html
<!DOCTYPE HTML>
<html>

<head>
  <meta charset="UTF-8">
  <title>Elm + Websockets</title>
  <script type="text/javascript" src="elm.js"></script>
</head>

<body>
	<div id="myapp"></div>
</body>

<script type="text/javascript">

// Start the Elm application.
var app = Elm.Main.init({
	node: document.getElementById('myapp')
});

// Create your WebSocket.
var socket = new WebSocket('wss://echo.websocket.org');

// When a command goes to the `sendMessage` port, we pass the message
// along to the WebSocket.
app.ports.sendMessage.subscribe(function(message) {
    socket.send(message);
});

// When a message comes into our WebSocket, we pass the message along
// to the `messageReceiver` port.
socket.addEventListener("message", function(event) {
	app.ports.messageReceiver.send(event.data);
});

// If you want to use a JavaScript library to manage your WebSocket
// connection, replace the code in JS with the alternate implementation.
</script>

</html>
```

با فراخوانی تابع `Elm.Main.init()` شروع کرده اما این بار از آبجکت `app` استفاده می‌کنیم. برای ارسال داده، از پورت `sendMessage` و برای دریافت داده، از پورت `messageReceiver` استفاده می‌کنیم.

این دو تابع، به کدی که در سمت Elm نوشته شده است، مربوط می‌شوند.

## کاربرد در Elm

به خطوطی که از کلمه کلیدی `port` در فایل Elm استفاده می‌کنند، نگاهی بیندازید. پورت‌هایی را که در سمت جاوا اسکریپت دیدیم، بدین صورت در Elm تعریف می‌کنیم:

```elm
port module Main exposing (..)

import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Json.Decode as D

-- MAIN

main : Program () Model Msg
main =
  Browser.element
    { init = init
    , view = view
    , update = update
    , subscriptions = subscriptions
    }

-- PORTS

port sendMessage : String -> Cmd msg
port messageReceiver : (String -> msg) -> Sub msg

-- MODEL

type alias Model =
  { draft : String
  , messages : List String
  }

init : () -> ( Model, Cmd Msg )
init flags =
  ( { draft = "", messages = [] }
  , Cmd.none
  )

-- UPDATE

type Msg
  = DraftChanged String
  | Send
  | Recv String

-- Use the `sendMessage` port when someone presses ENTER or clicks
-- the "Send" button. Check out index.html to see the corresponding
-- JS where this is piped into a WebSocket.
--
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
  case msg of
    DraftChanged draft ->
      ( { model | draft = draft }
      , Cmd.none
      )

    Send ->
      ( { model | draft = "" }
      , sendMessage model.draft
      )

    Recv message ->
      ( { model | messages = model.messages ++ [message] }
      , Cmd.none
      )

-- SUBSCRIPTIONS

-- Subscribe to the `messageReceiver` port to hear about messages coming in
-- from JS. Check out the index.html file to see how this is hooked up to a
-- WebSocket.
--
subscriptions : Model -> Sub Msg
subscriptions _ =
  messageReceiver Recv

-- VIEW

view : Model -> Html Msg
view model =
  div []
    [ h1 [] [ text "Eco Chat" ]
    , ul []
        (List.map (\msg -> li [] [ text msg ]) model.messages)
    , input
        [ type_ "text"
        , placeholder "Draft"
        , onInput DraftChanged
        , on "keydown" (ifIsEnter Send)
        , value model.draft
        ]
        []
    , button [ onClick Send ] [ text "Send" ]
    ]

-- DETECT ENTER

ifIsEnter : msg -> D.Decoder msg
ifIsEnter msg =
  D.field "key" D.string
    |> D.andThen (\key -> if key == "Enter" then D.succeed msg else D.fail "some other key")
```

توجه داشته باشید که در خط اول بجای `module` از `port module` استفاده شده است. این امکان وجود دارد که پورت‌ها را در یک ماژول خاص تعریف کنیم. در صورت نیاز، کامپایلر در این مورد شمارا راهنمایی می‌کند، بنابراین امیدواریم کسی در این مورد خیلی گیر نکند!

بسیار خوب، اما چه اتفاقی در تعریف `port` برای `sendMessage` و `messageReceiver` می‌افتد؟

## پیام‌های خروجی (`Cmd`)

تعریف تابع `sendMessage` اجازه می‌دهد تا پیام‌ها را از Elm ارسال کنیم.

```elm
port sendMessage : String -> Cmd msg
```

در اینجا اعلام می‌کنیم که می‌خواهیم مقدار `String` را ارسال کنیم، اما می‌توانیم هر نوع داده‌ای که با پرچم‌ها کار می‌کند را ارسال کنیم. در صفحه قبل درباره این نوع داده‌ها صحبت کردیم. می‌توانید به [این مثال `localStorage`](https://ellie-app.com/8yYddD6HRYJa1) نگاهی بیندازید تا ببینید یک [`Json.Encode.Value`](https://package.elm-lang.org/packages/elm/json/latest/Json-Encode#Value) چگونه به جاوا اسکریپت ارسال می‌شود.

در ادامه، می‌توانیم تابع `sendMessage` را مانند هر تابع دیگری فراخوانی کنیم. اگر تابع `update` یک دستور `sendMessage "hello"` صادر کند، در سمت جاوا اسکریپت درباره آن مطلع خواهید شد:

```javascript
app.ports.sendMessage.subscribe(function(message) {
    socket.send(message);
});
```

این کد جاوا اسکریپت به تمام پیام‌های خروجی مشترک شده است. می‌توانید چندین تابع را مشترک کنید و توابع را با استفاده از تکنیک فراخوانی با ارجاع، لغو اشتراک کنید. به‌طور کلی توصیه می‌کنیم که عملکرد آن را استاتیک یا ایستا نگه دارید.

همچنین توصیه می‌کنیم به جای اینکه تعداد زیادی پورت ایجاد کنید، پیام‌های غنی‌تری ارسال کنید. شاید این به معنای داشتن یک نوع داده سفارشی در Elm باشد که همه چیزهایی را که ممکن است بخواهید به جاوا اسرکیپت بگویید، نمایندگی کند و سپس از [`Json.Encode`](https://package.elm-lang.org/packages/elm/json/latest/Json-Encode) برای ارسال آن به یک اشتراک جاوا اسکریپت واحد استفاده کنید. بسیاری از توسعه‌دهندگان متوجه می‌شوند که این کار منجر به عملکرد تمیزتری از sepertion of concerns می‌شود. با استفاده از این تکنیک، Elm برخی از وضعیت‌ها را در اختیار دارد و جاوا اسکریپت وضعیت‌های دیگر را.

## پیام‌های ورودی (`Sub`)

تعریف تابع `messageReceiver` اجازه می‌دهد تا به پیام‌های ورودی Elm دسترسی یابیم.

```elm
port messageReceiver : (String -> msg) -> Sub msg
```

در اینجا می‌گوییم که قرار است مقدار `String` را دریافت کنیم، اما دوباره، می‌توانیم برای هر نوع داده‌ای که می‌تواند از طریق پرچم یا پورت خروجی وارد شود، آماده باشیم. فقط کافی است نوع داده `String` را با یکی از نوع داده‌هایی که می‌توانند از مرز عبور کنند، جایگزین کنید.

می‌توانیم تابع `messageReceiver` را مانند هر تابع دیگری فراخوانی کنیم. در این مورد، هنگام تعریف `subscriptions`، تابع `messageReceiver Recv` را فراخوانی می‌کنیم زیرا می‌خواهیم از هر پیام ورودی جاوا اسکریپت مطلع شویم. این کار به ما اجازه می‌دهد پیام‌هایی مانند `Recv "How are you?"` را در تابع `update` دریافت کنیم.

در سمت جاوا اسکریپت، می‌توانیم هر زمان که بخواهیم به این پورت چیزی ارسال کنیم:

```javascript
socket.addEventListener("message", function(event) {
	app.ports.messageReceiver.send(event.data);
});
```

به‌طور تصادفی هر بار که WebSocket یک پیام دریافت می‌کند، داده ارسال می‌کنیم. اما می‌توانید در زمان‌های دیگر نیز اینکار را انجام دهید. شاید پیام‌هایی از منبع داده دیگری نیز دریافت می‌کنیم. Elm نیازی به دانستن هیچ چیز درباره آن ندارد که این خوب است! فقط رشته‌های متنی را از طریق پورت مربوطه ارسال کنید.

## یادداشت

**کاربرد پورت در ایجاد مرزبندی قوی بین Elm و JavaScript است!** به هیچ وجه سعی نکنید برای هر تابع جاوا اسکریپت که نیاز دارید، یک پورت بسازید. ممکن است واقعا Elm را دوست داشته باشید و بخواهید همه چیز را در Elm انجام دهید، اما پورت‌ها برای اینکار طراحی نشده‌اند. در عوض، بر روی سوالاتی مانند "چه کسی مالک وضعیت فعلی است؟" تمرکز و از یک یا دو پورت برای ارسال پیام‌ها به‌طور متقابل استفاده کنید. اگر در یک سناریوی پیچیده هستید، می‌توانید حتی مقادیر `Msg` را با ارسال جاوا اسکریپت مانند `{ tag: "active-users-changed", list: ... }` شبیه‌سازی کنید که در آن یک برچسب برای تمام اطلاعاتی که ممکن است ارسال کنید، وجود دارد.

در ادامه، چند راهنمایی ساده و مشکلات رایج آمده است:

- **ارسال `Json.Encode.Value` از طریق پورت توصیه می‌شود.** مانند پرچم، برخی از نوع داده‌های اصلی نیز می‌توانند از طریق پورت عبور کنند. این عملکرد مربوط به زمانی است که هنوز دیکودرهای JSON وجود نداشتند که می‌توانید درباره آن بیشتر [مطالعه](https://elm-lang.org/docs/interop/flags.html#verifying-flags) کنید.

- **تمام تعریف‌های `port` باید در یک `port module` ظاهر شوند.** بهتر است تمام پورت‌های خود را در یک `port module` سازماندهی کنید تا مدیریت آن در یک فایل آسان‌تر شود.

- **پورت‌ها برای برنامه‌ها هستند.** یک `port module` فقط در برنامه‌ها در دسترس است، اما نه در بسته‌های نرم‌افزاری. اینکار اطمینان می‌دهد که توسعه‌دهندگان انعطاف‌پذیری لازم را در برنامه خود داشته باشند، اما اکوسیستم بسته‌های نرم‌افزاری به‌طور کامل با Elm شود. اعتقاد داریم، اینکار در دراز مدت یک اکوسیستم و جامعه کاربری قوی‌تر ایجاد خواهد کرد. در بخش بعدی درباره [محدودیت‌های تعامل Elm/JS](https://elm-lang.org/docs/interop/limits.html) می‌پردازیم.

- **پورت‌ها می‌توانند در فرآیند پاکسازی کد اضافی، حذف شوند.** فرآیند پاکسازی کد اضافی در Elm به نسبت تهاجمی است و پورت‌هایی که در کد Elm استفاده نمی‌شوند را حذف می‌کند. کامپایلر نمی‌داند در سمت جاوا اسکریپت چه می‌گذرد، بنابراین سعی کنید قبل از آن، چیزهای مرتبط را در Elm به یکدیگر متصل کنید.

امیدوارم این اطلاعات به شما کمک کند تا راهی برای گنجاندن Elm در JavaScript پروژه خود پیدا کنید! اینکار به اندازه انجام یک بازنویسی کامل در Elm جذاب نیست، اما تجربه نشان داده است که این استراتژی بسیار موثر است.