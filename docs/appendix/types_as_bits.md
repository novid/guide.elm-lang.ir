# نوع داده به عنوان بیت

در Elm، انواع مختلفی از داده وجود دارد:

- `Bool`
- `Int`
- `String`
- `(Int, Int)`
- `Maybe Int`
- ...

تا اینجا، درک مفهومی از آن‌ها داشتیم، اما یک کامپیوتر چگونه آن‌ها را درک می‌کند؟ `Maybe Int` چگونه بر روی هارد دیسک ذخیره می‌شود؟

## بیت

یک **بیت** جعبه کوچکی است که دو حالت دارد: صفر یا یک، خاموش یا روشن. حافظه اصلی کامپیوتر یک دنباله بسیار طولانی از بیت‌ها است.

بسیار خوب، فقط تعدادی بیت داریم که باید _همه چیز_ را با آن نمایندگی کنیم!

## `Bool`

یک مقدار `Bool` می‌تواند `True` یا `False` باشد. این دقیقا معادل یک بیت است!

## `Int`

یک مقدار `Int` عددی صحیح مانند `0`، `1`، `2` و غیره است. نمی‌توانید آن را در یک بیت جا دهید، بنابراین، گزینه دیگر استفاده از چندین بیت است. یک `Int` دنباله‌ای از چند بیت خواهد بود، مانند این:

```
00000000
00000001
00000010
00000011
...
```

می‌توانیم به طور دلخواه معنایی به هر یک از این دنباله‌ها اختصاص دهیم. بنابراین، شاید `00000000` صفر و `00000001` یک باشد. بسیار خوب! می‌توانیم فقط شروع به اختصاص شماره به دنباله‌های بیت به ترتیب صعودی کنیم. اما در نهایت، کم می‌آوریم...

با یک محاسبه سریع، هشت بیت فقط اجازه (2^8 = 256) عدد را می‌دهد. درباره اعدادی مانند مانند 9000 و 8004 چطور؟

پاسخ این است که فقط بیت‌های بیشتری اضافه کنیم. برای مدت طولانی، کامپیوترها از ۳۲ بیت استفاده می‌کردند. این اجازه می‌دهد تا (2^32 = 4,294,967,296) عدد، که انسان‌ها معمولا به آن فکر می‌کنند، پوشش داده شود. کامپیوترها این روزها از اعداد صحیح ۶۴ بیتی پشتیبانی می‌کنند، که اجازه می‌دهد (2^64 = 18,446,744,073,709,551,616) عدد پوشش داده شود. این خیلی زیاد است!

> **توجه:** اگر کنجکاو هستید که عملیات جمع چگونه کار می‌کند، درباره [مکمل دو](https://en.wikipedia.org/wiki/Two%27s_complement) یاد بگیرید. این فرآیند نشان می‌دهد که اعداد به طور دلخواه به دنباله‌های بیت اختصاص داده نمی‌شوند. به خاطر سرعت بخشیدن به عملیات جمع، این روش خاص اختصاص دادن اعداد واقعا خوب کار می‌کند.

## `String`

یک رشته متنی مانند `"abc"` دنباله‌ای از کاراکترهای `a` `b` `c` است، بنابراین، با تلاش برای ذخیره‌سازی کاراکتر به عنوان بیت شروع می‌کنیم.

یکی از روش‌های اولیه کدگذاری کاراکترها به نام [ASCII](https://en.wikipedia.org/wiki/ASCII) شناخته می‌شود. درست مانند اعداد صحیح، تصمیم گرفته شد که یک دنباله بیت را فهرست و شروع به اختصاص دادن مقادیر به طور دلخواه کنند:

```
00000000
00000001
00000010
00000011
...
```

بنابراین، هر کاراکتر باید در هشت بیت جا بگیرد، به این معنی که فقط ۲۵۶ کاراکتر می‌تواند نمایندگی شود! اگر فقط به زبان انگلیسی اهمیت می‌دهید، این روش واقعا خوب کار می‌کند. باید ۲۶ حرف کوچک، ۲۶ حرف بزرگ و ۱۰ عدد را پوشش دهید. در مجموع، ۶۲ عدد می شود. فضای زیادی برای نمادها و چیزهای عجیب و غریب دیگر باقیمانده است. در نهایت، [از این صفحه](https://ascii.cl/) می‌توانید ببینید که چه چیزی بدست آمده است .

اکنون ایده‌ای برای کاراکترها داریم، اما کامپیوتر چگونه می‌داند که `String` کجا تمام و قطعه بعدی داده کجا شروع می‌شود؟ همه چیز فقط بیت است. کاراکترها واقعا شبیه مقادیر `Int` به نظر می‌رسند! بنابراین، به راهی برای علامت‌گذاری پایان رشته متنی نیاز داریم.

این روزها، زبان‌های برنامه‌نویسی معمولا این کار را با ذخیره‌سازی **طول** رشته انجام می‌دهند. یک رشته مانند `"hello"` ممکن است چیزی شبیه به `5` `h` `e``l` `l` `o` در حافظه اصلی به نظر برسد. بنابراین، می‌دانید که یک `String` همیشه با ۳۲ بیت که طول را نمایندگی می‌کند شروع می‌شود. طول رشته متنی ۰ یا ۹۰۰۰ باشد، دقیقا می‌دانید که کاراکترهای آن کجا تمام می‌شوند.

> **توجه:** در یک مقطع، نیاز به پوشش زبانی بجز انگلیسی احساس شد. این تلاش در نهایت منجر به کدگذاری [UTF-8](https://en.wikipedia.org/wiki/UTF-8) شد. این کدگذاری، بسیار هوشمندانه است و شما را تشویق می‌کنم که درباره آن یاد بگیرید. معلوم می‌شود که "به دست آوردن کاراکتر پنجم" سخت‌تر از آن چیزی است که به نظر می‌رسد!

## `(Int, Int)`

درباره تاپل چطور؟ خوب، `(Int, Int)` شامل دو مقدار `Int` است و هر کدام یک دنباله از بیت‌ها هستند. بیایید این دو دنباله را در کنار هم در حافظه اصلی قرار دهیم و کار را تمام کنیم!

## نوع داده سفارشی

یک نوع داده سفارشی، درباره ترکیب انواع داده مختلف است. این انواع داده مختلف ممکن است اشکال متفاوتی داشته باشند. با نوع داده سفارشی `Color` شروع می‌کنیم:

```elm
type Color = Red | Yellow | Green
```

می‌توانیم به هر مورد یک عدد اختصاص دهیم: `Red = 0`، `Yellow = 1` و `Green = 2`. حالا می‌توانیم از `Int` استفاده کنیم. در اینجا، فقط به دو بیت نیاز داریم تا تمام موارد ممکن را پوشش دهیم، بنابراین `00` قرمز، `01` زرد، `10` سبز و `11` استفاده نشده است.

اما درباره نوع داده سفارشی که داده‌های اضافی را نگه می‌دارد، مانند `Maybe Int`، چطور؟ رویکرد معمول این است که مقداری بیت را برای "برچسب‌گذاری" داده‌ها کنار بگذاریم، بنابراین می‌توانیم تصمیم بگیریم که `Nothing = 0` و `Just = 1` باشد. در ادامه، چند نمونه آورده شده است:

- `Nothing` = `0`
- `Just 12` = `1` `00001100`
- `Just 16` = `1` `00010000`

یک عبارت `case`، قبل از اینکه تصمیم بگیرد چه کاری انجام دهد، همیشه به آن "برچسب" نگاه می‌کند. اگر یک `0` ببیند، می‌داند که داده‌ای وجود ندارد. اگر یک `1` ببیند، می‌داند که در ادامه آن یک دنباله از بیت‌ها وجود دارد که نماینده داده است.

این ایده "برچسب‌گذاری" مشابه قرار دادن طول در ابتدای مقادیر `String` است. مقادیر ممکن است اندازه‌های مختلفی داشته باشند، اما کد همیشه می‌تواند بفهمد که آن‌ها از کجا شروع و در کجا تمام می‌شوند.

## خلاصه

در نهایت، همه مقادیر باید توسط بیت‌ها نمایندگی شوند. این صفحه، یک نمای کلی از چگونگی کارکرد واقعی آن را ارایه می‌دهد.

معمولا، دلیلی برای فکر کردن به این موضوع وجود ندارد، اما آن را مفید یافتم تا درک خود را از نوع داده سفارشی و عبارت `case` عمیق‌تر کنم. امیدوارم برای شما نیز مفید باشد!

> **توجه:** اگر فکر می‌کنید این موضوع جالب است، ممکن است یادگیری بیشتر درباره Garbage Collection سرگرم‌کننده باشد. [کتابچه Garbage Collection](http://gchandbook.org/) را منبعی عالی در این زمینه یافته‌ام!