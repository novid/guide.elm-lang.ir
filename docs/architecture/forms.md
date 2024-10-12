# فرم‌ها

اکنون یک فرم ابتدایی خواهیم ساخت. این فرم دارای یک فیلد برای نام، یک فیلد برای گذرواژه و یک فیلد برای تایید گذرواژه است. همچنین برخی از اعتبارسنجی‌های بسیار ساده را انجام خواهیم داد تا بررسی کنیم آیا گذرواژه‌ها مطابقت دارند یا خیر.

در ادامه، کد برنامه قرار دارد. روی دکمه آبی "ویرایش" کلیک تا در ویرایشگر آنلاین با آن کار کنید. سعی کنید یک اشتباه تایپی وارد کنید تا برخی از پیام‌های خطا را ببینید. سعی کنید یک فیلد رکورد مانند `password` یا یک تابع مانند `placeholder` را به اشتباه بنویسید. **اکنون روی دکمه آبی کلیک کنید!**

<div class="edit-link"><a href="https://elm-lang.org/examples/forms">ویرایش</a></div>

```elm
import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (onInput)



-- MAIN


main =
  Browser.sandbox { init = init, update = update, view = view }



-- MODEL


type alias Model =
  { name : String
  , password : String
  , passwordAgain : String
  }


init : Model
init =
  Model "" "" ""



-- UPDATE


type Msg
  = Name String
  | Password String
  | PasswordAgain String


update : Msg -> Model -> Model
update msg model =
  case msg of
    Name name ->
      { model | name = name }

    Password password ->
      { model | password = password }

    PasswordAgain password ->
      { model | passwordAgain = password }



-- VIEW


view : Model -> Html Msg
view model =
  div []
    [ viewInput "text" "Name" model.name Name
    , viewInput "password" "Password" model.password Password
    , viewInput "password" "Re-enter Password" model.passwordAgain PasswordAgain
    , viewValidation model
    ]


viewInput : String -> String -> String -> (String -> msg) -> Html msg
viewInput t p v toMsg =
  input [ type_ t, placeholder p, value v, onInput toMsg ] []


viewValidation : Model -> Html msg
viewValidation model =
  if model.password == model.passwordAgain then
    div [ style "color" "green" ] [ text "OK" ]
  else
    div [ style "color" "red" ] [ text "Passwords do not match!" ]
```

این برنامه بسیار شبیه به [مثال فیلد متنی ](text_fields.md) است اما با فیلدهای بیشتری.

## Model

من همیشه با حدس زدن در مورد `Model` شروع می‌کنم. می‌دانیم که قرار است سه فیلد متنی داشته باشیم، بنابراین بیایید با همین پیش برویم:

```elm
type alias Model =
  { name : String
  , password : String
  , passwordAgain : String
  }
```

معمولا سعی می‌کنم با یک مدل حداقلی شروع کنم، شاید با فقط یک فیلد. سپس سعی می‌کنم توابع `view` و `update` را بنویسم. این اغلب نشان می‌دهد که نیاز دارم چیزهای بیشتری به `Model` اضافه کنم. ساختن تدریجی مدل به این معنی است که می‌توانم در طول فرآیند توسعه یک برنامه کارا داشته باشم. ممکن است هنوز تمام ویژگی‌ها را نداشته باشد، اما در حال پیشرفت است!


## Update

گاهی اوقات ایده خوبی از کد این تابع دارید. باید بتوانیم سه فیلد را تغییر دهیم، بنابراین به پیام‌هایی برای هر مورد نیاز داریم.

```elm
type Msg
  = Name String
  | Password String
  | PasswordAgain String
```

به این معنی است که تابع `update` به یک `case` برای هر سه نوع نیاز دارد:

```elm
update : Msg -> Model -> Model
update msg model =
  case msg of
    Name name ->
      { model | name = name }

    Password password ->
      { model | password = password }

    PasswordAgain password ->
      { model | passwordAgain = password }
```

هر مورد از شیوه بروزرسانی رکورد استفاده می‌کند تا اطمینان حاصل کند که فیلد مناسب تغییر می‌کند. این مشابه مثال قبلی است، به جز اینکه موارد بیشتری دارد.

اما در تابع `view` جزییات بیشتری را باید نمایش دهیم.


## View

این تابع `view` از **توابع کمکی** برای سازماندهی بهتر کد استفاده می‌کند:

```elm
view : Model -> Html Msg
view model =
  div []
    [ viewInput "text" "Name" model.name Name
    , viewInput "password" "Password" model.password Password
    , viewInput "password" "Re-enter Password" model.passwordAgain PasswordAgain
    , viewValidation model
    ]
```

در مثال‌های قبلی، به‌طور مستقیم از `input` و `div` استفاده کردیم. چرا این کار را متوقف کردیم؟

نکته جالب HTML در Elm این است که `input` و `div` فقط توابع عادی هستند. آن‌ها (۱) یک لیست از ویژگی‌ها و (2) یک لیست از فرزندان خود را می‌پذیرند. **از آنجا که از توابع عادی Elm استفاده می‌کنیم، به تمام قدرت Elm برای کمک در ساخت view دسترسی داریم!** می‌توانیم کد تکراری را به توابع کمکی سفارشی تبدیل کنیم. یعنی همین کاری که در اینجا انجام می‌دهیم!

بنابراین تابع `view` سه فراخوانی به `viewInput` دارد:

```elm
viewInput : String -> String -> String -> (String -> msg) -> Html msg
viewInput t p v toMsg =
  input [ type_ t, placeholder p, value v, onInput toMsg ] []
```

به این معنی است که نوشتن `viewInput "text" "Name" "Bill" Name` در Elm به یک مقدار HTML مانند `<input type="text" placeholder="Name" value="Bill">` هنگام نمایش در صفحه تبدیل می‌شود.

ورودی چهارم یک فراخوانی به `viewValidation` است:

```elm
viewValidation : Model -> Html msg
viewValidation model =
  if model.password == model.passwordAgain then
    div [ style "color" "green" ] [ text "OK" ]
  else
    div [ style "color" "red" ] [ text "Passwords do not match!" ]
```

این تابع ابتدا دو گذرواژه را مقایسه می‌کند. اگر مطابقت داشته باشند، متن سبز و یک پیام مثبت دریافت می‌کنید. اگر مطابقت نداشته باشند، متن قرمز و یک پیام خطا دریافت می‌کنید.

این توابع کمکی شروع به نشان دادن مزایای داشتن کتابخانه HTML به عنوان کد عادی Elm می‌کنند. ما _می‌توانستیم_ تمام آن کد را در `view` خود قرار دهیم، اما ساختن توابع کمکی در Elm کاملا عادی است، حتی در کد نمایشی. "آیا درک کد سخت شده است؟ شاید بتوانم یک تابع کمکی بسازم!"

> **تمرین‌ها:** به این مثال در ویرایشگر آنلاین [اینجا](https://elm-lang.org/examples/forms) نگاه کنید. سعی کنید ویژگی‌های زیر را به تابع کمکی `viewValidation` اضافه کنید:
>
> - بررسی کنید که گذرواژه بیشتر از ۸ کاراکتر باشد.
> - اطمینان حاصل کنید که گذرواژه شامل حروف بزرگ، کوچک و کاراکترهای عددی باشد.
>
> از توابع موجود در ماژول [`String`](https://package.elm-lang.org/packages/elm/core/latest/String) برای این تمرین‌ها استفاده کنید!
>
> **هشدار:** قبل از اینکه شروع به ارسال درخواست‌های HTTP کنیم، باید خیلی بیشتر یاد بگیریم. قبل از اینکه خودتان امتحان کنید، به خواندن ادامه دهید تا به بخش HTTP برسید. این کار با راهنمایی مناسب به طور قابل توجهی آسان‌تر خواهد بود!
>
> **توجه:** به نظر می‌رسد تلاش برای ساخت کتابخانه‌های اعتبارسنجی عمومی چندان موفق نبوده است. فکر می‌کنم مشکل این است که بررسی‌ها معمولا بهتر است که با توابع عادی Elm انجام شوند. برخی آرگومان‌ها را بگیرید، یک `Bool` یا `Maybe` برگردانید. برای نمونه، چرا از یک کتابخانه برای بررسی اینکه آیا دو رشته برابرند استفاده کنیم؟ بنابراین تا جایی که می‌دانیم، ساده‌ترین کد از نوشتن منطق برای سناریوی خاص شما بدون هیچ گونه اضافات خاصی به دست می‌آید. بنابراین حتما قبل از اینکه تصمیم بگیرید به چیزی پیچیده‌تر نیاز دارید، این کار را امتحان کنید!