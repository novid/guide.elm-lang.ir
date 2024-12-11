# پورت

برای برقراری ارتباط بین Elm و جاوااسکریپت، از Port یا پورت استفاده می‌شود.

پورت برای کاربردهای [`localStorage`][localstorage] و [`WebSockets`][websockets] استفاده می‌شود. بیایید روی نمونه `WebSockets` تمرکز کنیم.

## پورت در JavaScript {#ports-in-js}

در این نمونه، تقریبا همان ساختار HTML را داریم که در برنامه‌های قبلی استفاده کرده‌ایم، اما با کمی کد اضافی جاوااسکریپت. با ایجاد یک اتصال به `wss://echo.websocket.org` هر چیزی را که به آن ارسال کنید، تکرار می‌کند. در [نمونه کد Ellie][ellie] می‌توانید ببینید این کار به ما اجازه می‌دهد تا اسکلت یک اتاق چت را بسازیم:

```html linenums="1"
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

## پورت در Elm {#ports-in-elm}

به خطوطی که از کلمه کلیدی `port` در فایل Elm استفاده می‌کنند، نگاهی بیندازید. پورت‌هایی را که در سمت جاوااسکریپت دیدیم، بدین صورت در Elm تعریف می‌کنیم:

```elm linenums="1"
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

توجه داشته باشید که در خط اول بجای `module` از `port module` استفاده شده است. این امکان وجود دارد که پورت‌ها را در یک ماژول خاص تعریف کنیم. در صورت نیاز، کامپایلر در این مورد شما را راهنمایی می‌کند، بنابراین امیدواریم کسی در این مورد خیلی گیر نکند!

بسیار خوب، اما چه اتفاقی در تعریف `port` برای `sendMessage` و `messageReceiver` می‌افتد؟

## پیام‌های خروجی (`Cmd`) {#outgoing-messages}

تعریف تابع `sendMessage` اجازه می‌دهد تا پیام‌ها را از Elm ارسال کنیم.

```elm
port sendMessage : String -> Cmd msg
```

در اینجا اعلام می‌کنیم که می‌خواهیم مقدار `String` را ارسال کنیم، اما می‌توانیم هر نوع داده‌ای که با پرچم کار می‌کند را ارسال کنیم. در صفحه قبل درباره این نوع داده‌ها صحبت کردیم. می‌توانید به [این نمونه `localStorage`][ellie-localstorage] نگاهی بیندازید تا ببینید یک [`Json.Encode.Value`][json.encode-value] چگونه به جاوااسکریپت ارسال می‌شود.

در ادامه، می‌توانیم تابع `sendMessage` را مانند هر تابع دیگری فراخوانی کنیم. اگر تابع `update` یک دستور `sendMessage "hello"` صادر کند، در سمت جاوااسکریپت درباره آن مطلع خواهید شد:

```javascript
app.ports.sendMessage.subscribe(function(message) {
    socket.send(message);
});
```

این کد جاوااسکریپت به تمام پیام‌های خروجی مشترک شده است. می‌توانید چندین تابع را مشترک کنید و توابع را با استفاده از تکنیک فراخوانی با ارجاع، لغو اشتراک کنید. بطور کلی توصیه می‌کنیم که عملکرد آن را استاتیک یا ایستا نگه دارید.

همچنین توصیه می‌کنیم به جای اینکه تعداد زیادی پورت ایجاد کنید، پیام‌های غنی‌تری ارسال کنید. شاید این به معنای داشتن یک نوع داده سفارشی در Elm باشد که همه چیزهایی را که ممکن است بخواهید به جاوااسکریپت بگویید، نمایندگی کند و سپس از ماژول [`Json.Encode`][json-encode] برای ارسال آن به یک اشتراک جاوااسکریپت استفاده کنید. بسیاری از توسعه‌دهندگان متوجه می‌شوند که این کار منجر به عملکرد تمیزتری از sepertion of concerns می‌شود. با استفاده از این تکنیک، Elm برخی از وضعیت‌ها را در اختیار دارد و جاوااسکریپت وضعیت‌های دیگر را.

## پیام‌های ورودی (`Sub`) {#incoming-messages}

تعریف تابع `messageReceiver` اجازه می‌دهد تا به پیام‌های ورودی Elm دسترسی یابیم.

```elm
port messageReceiver : (String -> msg) -> Sub msg
```

در اینجا می‌گوییم که قرار است مقدار `String` را دریافت کنیم، اما دوباره، می‌توانیم برای هر نوع داده‌ای که می‌تواند از طریق پرچم یا پورت خروجی وارد شود، آماده باشیم. فقط کافی است نوع داده `String` را با یکی از نوع داده‌هایی که می‌توانند از مرز عبور کنند، جایگزین کنید.

می‌توانیم تابع `messageReceiver` را مانند هر تابع دیگری فراخوانی کنیم. در این مورد، هنگام تعریف `subscriptions`، تابع `messageReceiver Recv` را فراخوانی می‌کنیم زیرا می‌خواهیم از هر پیام ورودی جاوااسکریپت مطلع شویم. این کار به ما اجازه می‌دهد پیام‌هایی مانند `Recv "How are you?"` را در تابع `update` دریافت کنیم.

در سمت جاوااسکریپت، می‌توانیم هر زمان که بخواهیم به این پورت چیزی ارسال کنیم:

```javascript
socket.addEventListener("message", function(event) {
	app.ports.messageReceiver.send(event.data);
});
```

بطور تصادفی هر بار که WebSocket یک پیام دریافت می‌کند، داده ارسال می‌کنیم. اما می‌توانید در زمان‌های دیگر نیز این کار را انجام دهید. شاید پیام‌هایی از منبع داده دیگری نیز دریافت می‌کنیم. Elm نیازی به دانستن هیچ چیز درباره آن ندارد، که این خوب است! فقط رشته‌های متنی را از طریق پورت مربوطه ارسال کنید.

!!! note "یادداشت"

	**کاربرد پورت در ایجاد مرزبندی قوی بین Elm و JavaScript است!** به هیچ وجه سعی نکنید برای هر تابع جاوااسکریپت که نیاز دارید، یک پورت بسازید. ممکن است واقعا Elm را دوست داشته باشید و بخواهید همه چیز را در Elm انجام دهید، اما پورت‌ها برای این کار طراحی نشده‌اند. در عوض، بر روی سوالاتی مانند "چه کسی مالک وضعیت فعلی است؟" تمرکز و از یک یا دو پورت برای ارسال پیام‌ها بطور متقابل استفاده کنید. اگر در یک سناریوی پیچیده هستید، می‌توانید حتی مقادیر `Msg` را با ارسال جاوااسکریپت مانند `{ tag: "active-users-changed", list: ... }` شبیه‌سازی کنید که در آن یک برچسب برای تمام اطلاعاتی که ممکن است ارسال کنید، وجود دارد.

	در ادامه، چند راهنمایی ساده و مشکلات رایج آمده است:

	- **ارسال `Json.Encode.Value` از طریق پورت توصیه می‌شود.** مانند پرچم، برخی از نوع داده‌های اصلی نیز می‌توانند از طریق پورت عبور کنند. این عملکرد مربوط به زمانی است که هنوز دیکودرهای JSON وجود نداشتند که می‌توانید درباره آن بیشتر [مطالعه کنید][verify-flags].

	- **تمام تعریف‌های `port` باید در یک `port module` ظاهر شوند.** بهتر است تمام پورت‌های خود را در یک `port module` سازماندهی کنید تا مدیریت آن در یک فایل آسان‌تر شود.

	- **پورت‌ها برای برنامه‌ها هستند.** یک `port module` فقط در برنامه‌ها در دسترس است، اما نه در بسته‌های Elm. این کار اطمینان می‌دهد که توسعه‌دهندگان انعطاف‌پذیری لازم را در برنامه خود داشته باشند، اما اکوسیستم بسته‌ها بطور کامل با Elm پیاده‌سازی شود. اعتقاد داریم، اینکار در دراز مدت یک اکوسیستم و جامعه کاربری قوی‌تر ایجاد خواهد کرد. در بخش بعدی به [محدودیت‌های تعامل با جاوااسکریپت](limits.md) می‌پردازیم.

	- **پورت‌ها می‌توانند در فرآیند پاکسازی کد اضافی، حذف شوند.** فرآیند پاکسازی کد اضافی در Elm به نسبت تهاجمی است و پورت‌هایی که در کد Elm استفاده نمی‌شوند را حذف می‌کند. کامپایلر نمی‌داند در سمت جاوااسکریپت چه می‌گذرد، بنابراین سعی کنید قبل از آن، چیزهای مرتبط را در Elm به یکدیگر متصل کنید.

	امیدوارم این اطلاعات به شما کمک کند تا راهی برای گنجاندن Elm در پروژه خود پیدا کنید! این کار به اندازه انجام یک بازنویسی کامل در Elm جذاب نیست، اما تجربه نشان داده است که این استراتژی بسیار موثر است.

[localstorage]: https://github.com/elm-community/js-integration-examples/tree/master/localStorage
[websockets]: https://github.com/elm-community/js-integration-examples/tree/master/websockets
[ellie]: https://ellie-app.com/8yYgw7y7sM2a1
[ellie-localstorage]: https://ellie-app.com/8yYddD6HRYJa1
[json.encode.value]: https://package.elm-lang.org/packages/elm/json/latest/Json-Encode#Value
[json-encode]: https://package.elm-lang.org/packages/elm/json/latest/Json-Encode
[verify-flags]: https://elm-lang.org/docs/interop/flags.html#verifying-flags