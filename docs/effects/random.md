# Random

تا کنون فقط دستورات مربوط به درخواست HTTP را دیده‌ایم. اما می‌توانیم دستورات دیگری، مانند تولید مقادیر تصادفی، نیز صادر کنیم! بنابراین، یک برنامه می‌سازیم که با پرتاب تاس، یک عدد تصادفی بین ۱ و ۶ تولید می‌کند.

برای دیدن این برنامه، روی دکمه آبی "ویرایش" کلیک کنید. چند عدد تصادفی تولید و به کد نگاه کنید تا ببینید چگونه کار می‌کند. **همین حالا روی دکمه آبی کلیک کنید!**

<div class="edit-link"><a href="https://elm-lang.org/examples/numbers">ویرایش</a></div>

```elm
import Browser
import Html exposing (..)
import Html.Events exposing (..)
import Random



-- MAIN


main =
  Browser.element
    { init = init
    , update = update
    , subscriptions = subscriptions
    , view = view
    }



-- MODEL


type alias Model =
  { dieFace : Int
  }


init : () -> (Model, Cmd Msg)
init _ =
  ( Model 1
  , Cmd.none
  )



-- UPDATE


type Msg
  = Roll
  | NewFace Int


update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
  case msg of
    Roll ->
      ( model
      , Random.generate NewFace (Random.int 1 6)
      )

    NewFace newFace ->
      ( Model newFace
      , Cmd.none
      )



-- SUBSCRIPTIONS


subscriptions : Model -> Sub Msg
subscriptions model =
  Sub.none



-- VIEW


view : Model -> Html Msg
view model =
  div []
    [ h1 [] [ text (String.fromInt model.dieFace) ]
    , button [ onClick Roll ] [ text "Roll" ]
    ]
```

موضوع جدیدی که در اینجا وجود دارد، دستوری است که در تابع `update` صادر می‌شود:

```elm
Random.generate NewFace (Random.int 1 6)
```

تولید مقادیر تصادفی، کمی متفاوت از زبان‌هایی مانند جاوا اسکریپت و پایتون است. بنابراین، بیایید ببینیم چگونه در Elm انجام می‌شود!

## تولیدکننده‌های تصادفی

ما از بسته [`elm/random`][readme] و ماژول [`Random`][random] برای این کار استفاده می‌کنیم.

[readme]: https://package.elm-lang.org/packages/elm/random/latest
[random]: https://package.elm-lang.org/packages/elm/random/latest/Random

ایده اصلی این است که یک `Generator` تصادفی داریم که توصیف می‌کند _چگونه_ یک مقدار تصادفی تولید شود. برای نمونه:

```elm
import Random

probability : Random.Generator Float
probability =
  Random.float 0 1

roll : Random.Generator Int
roll =
  Random.int 1 6

usuallyTrue : Random.Generator Bool
usuallyTrue =
  Random.weighted (80, True) [ (20, False) ]
```

بنابراین، در اینجا سه تولیدکننده تصادفی داریم. تولیدکننده `roll` می‌گوید که یک `Int` تولید خواهد کرد که بطور خاص، یک عدد صحیح بین `1` و `6` خواهد بود. به همین ترتیب، تولیدکننده `usuallyTrue` می‌گوید که یک `Bool` تولید خواهد کرد که بطور خاص، ۸۰٪ مواقع درست خواهد بود.

نکته این است که هنوز در حال تولید مقادیر نیستیم. فقط توصیف می‌کنیم _چگونه_ آن‌ها را تولید کنیم. در ادامه، از تابع [`Random.generate`][gen] برای تبدیل آن به یک دستور استفاده می‌کنیم:

```elm
generate : (a -> msg) -> Generator a -> Cmd msg
```

زمانی که دستور اجرا می‌شود، `Generator` مقداری تولید می‌کند که در ادامه به یک پیام برای تابع `update` تبدیل می‌شود. در مثال ما، `Generator` مقداری بین ۱ و ۶ تولید می‌کند و به یک پیام مانند `NewFace 1` یا `NewFace 4` تبدیل می‌شود. این تمام چیزی است که برای دریافت پرتاب‌های تصادفی تاس نیاز داریم، اما تولیدکننده‌ها می‌توانند کارهای بیشتری انجام دهند!

[gen]: https://package.elm-lang.org/packages/elm/random/latest/Random#generate

## ترکیب تولیدکننده‌ها

زمانی که تولیدکننده ساده‌ای مانند `probability` و `usuallyTrue` داریم، می‌توانیم شروع به ترکیب آن‌ها با تابعی مانند `map3` کنیم. تصور کنید می‌خواهیم یک دستگاه اسلات ساده بسازیم. می‌توانیم یک تولیدکننده به این شکل ایجاد کنیم:

```elm
import Random

type Symbol = Cherry | Seven | Bar | Grapes

symbol : Random.Generator Symbol
symbol =
  Random.uniform Cherry [ Seven, Bar, Grapes ]

type alias Spin =
  { one : Symbol
  , two : Symbol
  , three : Symbol
  }

spin : Random.Generator Spin
spin =
  Random.map3 Spin symbol symbol symbol
```

ابتدا `Symbol` را ایجاد می‌کنیم تا تصاویری که می‌توانند در دستگاه اسلات ظاهر شوند را توصیف کنیم. سپس یک تولیدکننده تصادفی ایجاد می‌کنیم که هر نماد را با احتمال برابر تولید می‌کند.

در ادامه، از تابع `map3` برای ترکیب آن‌ها به یک تولیدکننده جدید به نام `spin` استفاده می‌کنیم. این کار، بیان می‌کند که سه نماد تولید کند و آن ها را در یک `Spin` قرار دهد.

نکته این است که از بلوک‌های کوچک، می‌توانیم یک `Generator` ایجاد کنیم که رفتار نسبتا پیچیده‌ای را توصیف کند. سپس درون برنامه، برای دریافت مقدار تصادفی بعدی، فقط کافی است فراخوانی `Random.generate NewSpin spin` را انجام دهیم.


> **تمرین:** در ادامه، چند ایده برای جالب‌تر کردن برنامه قبل وجود دارد!
>
>   - بجای نمایش یک عدد، تصویر تاس را نشان دهید.
>   - بجای نمایش تصویر تاس، از بسته [`elm/svg`][svg] استفاده کنید تا خودتان آن را بکشید.
>   - یک تاس وزن‌دار با استفاده از تابع [`Random.weighted`][weighted] ایجاد کنید.
>   - یک تاس دوم اضافه کنید و بگذارید هر دو به طور همزمان پرتاب شوند.
>   - بگذارید تاس‌ها به طور تصادفی بچرخند قبل از اینکه روی یک مقدار نهایی ثابت شوند.

[svg]: https://package.elm-lang.org/packages/elm/svg/latest/
[weighted]: https://package.elm-lang.org/packages/elm/random/latest/Random#weighted