# فیلدهای متنی

می‌خواهیم یک برنامه ساده بنویسیم که محتوای یک فیلد متنی را معکوس می‌کند.

برای مشاهده این برنامه در ویرایشگر آنلاین، روی دکمه آبی کلیک کنید. سعی کنید نکته مربوط به کلمه کلیدی `type` را بررسی کنید. **اکنون روی دکمه آبی کلیک کنید!**

<div class="edit-link"><a href="https://elm-lang.org/examples/text-fields">ویرایش</a></div>

```elm
import Browser
import Html exposing (Html, Attribute, div, input, text)
import Html.Attributes exposing (..)
import Html.Events exposing (onInput)



-- MAIN


main =
  Browser.sandbox { init = init, update = update, view = view }



-- MODEL


type alias Model =
  { content : String
  }


init : Model
init =
  { content = "" }



-- UPDATE


type Msg
  = Change String


update : Msg -> Model -> Model
update msg model =
  case msg of
    Change newContent ->
      { model | content = newContent }



-- VIEW


view : Model -> Html Msg
view model =
  div []
    [ input [ placeholder "Text to reverse", value model.content, onInput Change ] []
    , div [] [ text (String.reverse model.content) ]
    ]
```

این کد یک تغییر جزیی از مثال قبلی است. پس از مدل‌سازی اولیه داده، چند پیام تعریف می‌کنید. تعیین می‌کنید که چگونه تابع `update` اجرا شود و در ادامه تابع `view` را می‌سازید. تفاوت در شیوه مدل‌سازی داده، نمایش محتوا و پاسخ به پیام‌های دریافتی است. بیایید به توضیح آن بپردازیم!

## Model

من همیشه با حدس زدن اینکه مدل من باید چه باشد شروع می‌کنم. می‌دانیم که باید هر چیزی را که کاربر در فیلد متنی تایپ کرده است، پیگیری کنیم. به این اطلاعات نیاز داریم تا بدانیم چگونه متن معکوس شده را نمایش دهیم. بنابراین با این مدل پیش می‌رویم:

```elm
type alias Model =
  { content : String
  }
```

این بار تصمیم گرفتم مدل را به عنوان یک رکورد نمایش دهم. رکورد، ورودی کاربر را در فیلد `content` ذخیره می‌کند.

> **توجه:** ممکن است بپرسید، چرا باید رکورد داشته باشیم اگر فقط یک ورودی وجود دارد؟ آیا نمی‌شد از رشته متنی به طور مستقیم استفاده کرد؟ البته که می‌شد! اما شروع با یک رکورد اضافه کردن فیلدهای بیشتر را به راحتی امکان‌پذیر می‌کند، زیرا برنامه در ادامه پیچیده‌تر می‌شود. وقتی زمان آن برسد که بخواهیم *دو* ورودی متنی داشته باشیم، کد کمتری را دستکاری می‌کنیم.

## View

پس از مدل‌سازی داده، معمولا با ایجاد یک تابع `view` ادامه می‌دهم:

```elm
view : Model -> Html Msg
view model =
  div []
    [ input [ placeholder "Text to reverse", value model.content, onInput Change ] []
    , div [] [ text (String.reverse model.content) ]
    ]
```

یک `<div>` با دو فرزند ایجاد می‌کنیم. فرزند `<input>` مورد نظر ماست که سه ویژگی دارد:

- `placeholder` متنی است که وقتی محتوایی وجود ندارد نمایش داده می‌شود
- `value` محتوای فعلی این `<input>` است
- `onInput` پیام‌ها را زمانی که کاربر در این `<input>` تایپ می‌کند ارسال می‌کند

تایپ کردن "bard" این چهار پیام را تولید می‌کند:

1. `Change "b"`
2. `Change "ba"`
3. `Change "bar"`
4. `Change "bard"`

این پیام‌ها به تابع `update` ارسال می‌شوند.

## Update

در این برنامه فقط یک نوع پیام وجود دارد، بنابراین تابع `update` فقط باید یک حالت را مدیریت کند:

```elm
type Msg
  = Change String

update : Msg -> Model -> Model
update msg model =
  case msg of
    Change newContent ->
      { model | content = newContent }
```

هنگام دریافت پیام مبنی بر تغییر `<input>`، محتوای مدل را بروزرسانی می‌کنیم. بنابراین اگر "bard" را تایپ کنید، پیام‌های دریافتی، مدل‌های زیر را تولید می‌کنند:

1. `{ content = "b" }`
2. `{ content = "ba" }`
3. `{ content = "bar" }`
4. `{ content = "bard" }`

ما باید این اطلاعات را به طور صریح در مدل خود پیگیری کنیم، در غیر این صورت هیچ راهی برای نمایش متن معکوس شده در تابع `view` نخواهیم داشت!

> **تمرین:** به مثال در ویرایشگر آنلاین [اینجا](https://elm-lang.org/examples/text-fields) بروید و طول `content` را در تابع `view` نمایش دهید. از تابع [`String.length`](https://package.elm-lang.org/packages/elm/core/latest/String#length) استفاده کنید!
> **توجه:** اگر می‌خواهید اطلاعات بیشتری درباره کارکرد مقادیر `Change` در این برنامه بدانید، فصل‌های مربوط به [نوع داده سفارشی](/types/custom_types.html) و [تطبیق الگو](/types/pattern_matching.html) را مطالعه کنید.