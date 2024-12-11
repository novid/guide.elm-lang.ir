# Result

نوع داده `Maybe` می‌تواند در توابع ساده‌ای که ممکن است شکست بخورند مفید باشد، اما به شما نمی‌گوید _چرا_ شکست خورده است. تصور کنید که یک کامپایلر، هنگام بروز اشتباه در برنامه، فقط بگوید `Nothing`. ببینم چطور می‌خواهید اشتباه را پیدا کنید!

اینجاست که نوع داده [`Result`][result] بکار می‌آید. این نوع، به صورت زیر تعریف شده است:

```elm
type Result error value
  = Ok value
  | Err error
```

هدف از این نوع داده، ارایه اطلاعات اضافی در زمان بروز خطا است. این حالت، برای گزارش خطا و بازیابی خطا بسیار مفید است!

## گزارش خطا {#error-reporting}

شاید وبسایتی داشته باشیم که در آن کاربران سن خود را وارد می‌کنند. برای بررسی معقول بودن ورودی کاربر، می‌توانیم از تابعی مانند این استفاده کنیم:

```elm
isReasonableAge : String -> Result String Int
isReasonableAge input =
  case String.toInt input of
    Nothing ->
      Err "That is not a number!"

    Just age ->
      if age < 0 then
        Err "Please try again after you are born."

      else if age > 135 then
        Err "Are you some kind of turtle?"

      else
        Ok age

-- isReasonableAge "abc" == Err ...
-- isReasonableAge "-13" == Err ...
-- isReasonableAge "24"  == Ok 24
-- isReasonableAge "150" == Err ...
```

نه تنها می‌توانیم سن را بررسی کنیم، بلکه پیام‌های خطا را بسته به جزییات ورودی نشان می‌دهیم. این نوع بازخورد بسیار بهتر از `Nothing` است!

## بازیابی خطا {#error-recovery}

نوع داده `Result` همچنین می‌تواند در بازیابی از خطاها کمک کند. یکی از جاهایی که این را می‌بینید، هنگام انجام درخواست‌های HTTP است. فرض کنید می‌خواهیم متن کامل _آنا کارنینا_ اثر لئو تولستوی را نشان دهیم. نتیجه درخواست HTTP یک `Result Error String` است تا این واقعیت را که درخواست ممکن است با متن کامل موفق شود یا ممکن است به روش‌های مختلفی شکست بخورد، ثبت کند:

```elm
type Error
  = BadUrl String
  | Timeout
  | NetworkError
  | BadStatus Int
  | BadBody String

-- Ok "All happy ..." : Result Error String
-- Err Timeout        : Result Error String
-- Err NetworkError   : Result Error String
```

از آنجا می‌توانیم پیام‌های خطای بهتری را که قبلا بحث کردیم نشان دهیم، اما همچنین می‌توانیم سعی کنیم خطا را بازیابی کنیم! اگر یک `Timeout` ببینیم، ممکن است با کمی صبر و تلاش دوباره، برطرف شد. در حالی که اگر یک `BadStatus 404` ببینیم، دیگر هیچ دلیلی برای تلاش دوباره وجود ندارد.

فصل بعدی نشان می‌دهد که چگونه درخواست‌های HTTP را انجام دهیم، بنابراین بزودی با نوع داده `Result` و `Error` روبرو خواهیم شد!

[result]: https://package.elm-lang.org/packages/elm-lang/core/latest/Result#Result