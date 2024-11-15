# تجزیه و تحلیل URL

در یک وب اپلیکیشن واقعی، می‌خواهیم محتوای متفاوتی را برای URLهای مختلف نمایش دهیم:

- `/search`
- `/search?q=seiza`
- `/settings`

این کار چگونه انجام می‌شود؟ از بسته [`elm/url`](https://package.elm-lang.org/packages/elm/url/latest/) برای تجزیه و تحلیل رشته‌های خام به ساختار داده‌های زیبا در Elm استفاده می‌کنیم. درک عملکرد این بسته ممکن است در ابتدا کمی پیچیده باشد، بنابراین با ارایه مثال‌های مختلف آن را توضیح می‌دهیم. این دقیقا کاری است که در ادامه انجام خواهیم داد!

## مثال اول

فرض کنید یک وبسایت هنری داریم که شامل آدرس‌های زیر می‌شود:

- `/topic/architecture`
- `/topic/painting`
- `/topic/sculpture`
- `/blog/42`
- `/blog/123`
- `/blog/451`
- `/user/tom`
- `/user/sue`
- `/user/sue/comment/11`
- `/user/sue/comment/51`

در این مورد، صفحات موضوع، پست‌های وبلاگ، اطلاعات کاربری و راهی برای جستجوی نظرات کاربران داریم. از ماژول [`Url.Parser`](https://package.elm-lang.org/packages/elm/url/latest/Url-Parser) برای نوشتن یک تحلیل‌گر URL استفاده می‌کنیم:

```elm
import Url.Parser exposing (Parser, (</>), int, map, oneOf, s, string)

type Route
  = Topic String
  | Blog Int
  | User String
  | Comment String Int

routeParser : Parser (Route -> a) a
routeParser =
  oneOf
    [ map Topic   (s "topic" </> string)
    , map Blog    (s "blog" </> int)
    , map User    (s "user" </> string)
    , map Comment (s "user" </> string </> s "comment" </> int)
    ]

-- /topic/pottery        ==>  Just (Topic "pottery")
-- /topic/collage        ==>  Just (Topic "collage")
-- /topic/               ==>  Nothing

-- /blog/42              ==>  Just (Blog 42)
-- /blog/123             ==>  Just (Blog 123)
-- /blog/mosaic          ==>  Nothing

-- /user/tom/            ==>  Just (User "tom")
-- /user/sue/            ==>  Just (User "sue")
-- /user/bob/comment/42  ==>  Just (Comment "bob" 42)
-- /user/sam/comment/35  ==>  Just (Comment "sam" 35)
-- /user/sam/comment/    ==>  Nothing
-- /user/                ==>  Nothing
```

ماژول `Url.Parser` این امکان را فراهم می‌کند که بطور مختصر URLهای معتبر را به ساختار داده‌های زیبا در Elm تبدیل کنیم!

## مثال دوم

فرض کنید یک وبلاگ شخصی داریم که شامل آدرس‌های زیر می‌شود:

- `/blog/12/the-history-of-chairs`
- `/blog/13/the-endless-september`
- `/blog/14/whale-facts`
- `/blog/`
- `/blog?q=whales`
- `/blog?q=seiza`

در این مورد، صفحات پست‌های وبلاگ فردی و یک نمای کلی از وبلاگ با یک پارامتر جستجوی اختیاری داریم. از ماژول [`Url.Parser.Query`](https://package.elm-lang.org/packages/elm/url/latest/Url-Parser-Query) برای نوشتن یک تحلیل‌گر URL استفاده می‌کنیم:

```elm
import Url.Parser exposing (Parser, (</>), (<?>), int, map, oneOf, s, string)
import Url.Parser.Query as Query

type Route
  = BlogPost Int String
  | BlogQuery (Maybe String)

routeParser : Parser (Route -> a) a
routeParser =
  oneOf
    [ map BlogPost  (s "blog" </> int </> string)
    , map BlogQuery (s "blog" <?> Query.string "q")
    ]

-- /blog/14/whale-facts  ==>  Just (BlogPost 14 "whale-facts")
-- /blog/14              ==>  Nothing
-- /blog/whale-facts     ==>  Nothing
-- /blog/                ==>  Just (BlogQuery Nothing)
-- /blog                 ==>  Just (BlogQuery Nothing)
-- /blog?q=chabudai      ==>  Just (BlogQuery (Just "chabudai"))
-- /blog/?q=whales       ==>  Just (BlogQuery (Just "whales"))
-- /blog/?query=whales   ==>  Just (BlogQuery Nothing)
```

عملگرهای `</>` و `<?>` این امکان را فراهم می‌کنند تا کدی بنویسیم که بطور قابل توجهی شبیه به URLهای واقعی باشد که می‌خواهیم آن‌ها را تجزیه و تحلیل کنیم. افزودن ماژول `Url.Parser.Query` به ما اجازه داد تا پارامترهای جستجو مانند `?q=seiza` را مدیریت کنیم.

## مثال سوم

فرض کنید یک وبسایت مستندات فنی داریم که شامل آدرس‌های زیر می‌شود:

- `/Basics`
- `/Maybe`
- `/List`
- `/List#map`
- `/List#filter`
- `/List#foldl`

در این مورد، می‌توانیم از تابع [`fragment`](https://package.elm-lang.org/packages/elm/url/latest/Url-Parser#fragment) در ماژول `Url.Parser` برای مدیریت این آدرس‌ها استفاده کنیم:

```elm
type alias Docs =
  (String, Maybe String)

docsParser : Parser (Docs -> a) a
docsParser =
  map Tuple.pair (string </> fragment identity)

-- /Basics     ==>  Just ("Basics", Nothing)
-- /Maybe      ==>  Just ("Maybe", Nothing)
-- /List       ==>  Just ("List", Nothing)
-- /List#map   ==>  Just ("List", Just "map")
-- /List#      ==>  Just ("List", Just "")
-- /List/map   ==>  Nothing
-- /           ==>  Nothing
```

اکنون می‌توانیم URL Fragment را نیز مدیریت کنیم!

## حالت ترکیبی

اکنون که چند نمونه را مشاهده کردیم، باید ببینیم که این عملکرد چگونه در یک برنامه `Browser.application` قرار می‌گیرد. بجای اینکه فقط URL فعلی را مانند دفعه قبل ذخیره‌سازی کنیم، آیا می‌توانیم آن را به داده‌های مفید تجزیه کنیم و بجای آن نمایش دهیم؟

```elm
TODO
```

مفاهیم جدید این قسمت عبارتند از:

1. تابع `update`، هنگام دریافت یک پیام `UrlChanged`، مقدار URL را تجزیه و تحلیل می‌کند.
2. تابع `view`، محتوای متفاوتی را برای آدرس‌های مختلف نمایش می‌دهد.

بسیار خوب، واقعا خیلی پیچیده نیست!

اما چه اتفاقی می‌افتد وقتی که ۱۰، ۲۰ یا ۱۰۰ صفحه مختلف دارید؟ آیا همه آن‌ها در یک تابع `view` قرار می‌گیرند؟ همه این صفحه‌ها که نمی‌توانند در یک فایل قرار گیرند. چند فایل باید ایجاد شود؟ ساختار دایرکتوری باید چگونه باشد؟ این چیزی است که در ادامه بحث خواهیم کرد!