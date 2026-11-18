# School Faghahat Crawler (JSON Export Only)

این نسخه فقط برای Crawl و ذخیره خروجی محلی است. در آن هیچ کد یا وابستگی Elasticsearch و هیچ سرویس Embedding وجود ندارد.

## خروجی

برای هر کتاب یک پوشه ساخته می‌شود:

```text
output/
  books/
    1000000/
      book.json
      manifest.json
      volume_001.jsonl
      volume_002.jsonl
```

- `book.json`: مشخصات کتاب و فهرست مطالب استخراج‌شده.
- `manifest.json`: وضعیت آخرین اجرا، جلدها، فایل‌های خروجی و خطاها.
- `volume_NNN.jsonl`: هر خط یک Chunk مستقل به همراه شناسه، شناسه کتاب، جلد، صفحه، ترتیب صفحه و ترتیب Chunk است.
- `data/crawler.sqlite3`: رجیستری کتاب‌ها و وضعیت صفحات برای ادامه Crawl پس از توقف.

شناسه داخلی کتاب از `1000000` شروع می‌شود و در SQLite پایدار می‌ماند. شناسه Chunk نیز بر اساس شناسه کتاب، جلد، ترتیب صفحه و ترتیب Chunk ساخته می‌شود.

## نصب

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```
##  آخرین ایدی 
```bash
ایدی بعدی آخرین کتاب 
1000330
```

## آخرین موضوعات 
```bash
 "تاریخ و جغرافیا" 
 "سیره" 
 "اخلاق"
```

## اجرا

ابتدا کتاب‌های قابل کشف را شناسایی کن:

```bash
python -m app.main discover
python -m app.main list-books
```

ابتدا فقط یک کتاب را آزمایشی Crawl کن:

```bash
python -m app.main crawl --source-book-id 26395
```

فقط یک جلد:

```bash
python -m app.main crawl --source-book-id 26395 --volume 1
```

ادامه Crawl کتابی که قبلاً شروع شده:

```bash
python -m app.main resume --source-book-id 26395
```

پس از بررسی خروجی چند کتاب، برای Crawl دسته‌ای در `.env` مقدار زیر را فعال کن:

```env
CRAWL_ALL_ENABLED=true
```

سپس:

```bash
python -m app.main crawl-all
# یا محدود به 10 کتاب
python -m app.main crawl-all --limit 10
```

برای Crawl بر اساس دسته‌بندی موضوعی (با تفکیک کامل کتاب‌ها):

```bash
# بر اساس آدرس صفحه دسته‌بندی
python -m app.main crawl-category --category "https://lib.eshia.ir/سیاسی/all"

# یا بر اساس نام دسته‌بندی
python -m app.main crawl-category --category "سیاسی"
python -m app.main crawl-category --category "علوم عقلی"
python -m app.main crawl-category --category "تاریخ و جغرافیا" "سیره" "اخلاق"

# یا محدود کردن به تعداد مشخصی از کتاب‌های آن دسته
python -m app.main crawl-category --category "سیاسی" --limit 5
```

## نکات مهم

1. این نسخه فقط فایل JSON/JSONL تولید می‌کند؛ نه به Elasticsearch وصل می‌شود و نه Embedding می‌سازد.
2. برای جلوگیری از درخواست‌های پیاپی، بین درخواست‌ها تأخیر وجود دارد و `robots.txt` رعایت می‌شود.
3. ساختار HTML سایت ممکن است بین کتاب‌ها متفاوت باشد. پیش از اجرای سراسری، خروجی چند کتاب و چند جلد را بررسی کن و در صورت نیاز Selectorها را در `.env` تنظیم کن.
4. اگر صفحه‌ای خطا بدهد، وضعیت آن در SQLite ثبت می‌شود و اجرای بعدی آن را دوباره امتحان می‌کند.
5. فایل‌های JSONL به‌صورت append نوشته می‌شوند؛ صفحه‌ای که با موفقیت ثبت شده در اجرای بعدی تکرار نمی‌شود. اگر خروجی یک کتاب را دستی پاک کردی، وضعیت همان صفحات را نیز از SQLite پاک کن یا فایل SQLite را پس از پشتیبان‌گیری اصلاح کن.
6. کشف خودکار کتاب‌ها به لینک‌هایی بستگی دارد که از صفحه شروع در دسترس‌اند؛ کامل بودن فهرست کتاب‌ها و صحت استخراج تمام صفحات تا زمان اعتبارسنجی روی HTML واقعی تضمین‌شده نیست.

## قالب هر خط JSONL

هر خط یک سند Chunk است و برای مرحله بعدی Import/Embedding قابل استفاده خواهد بود. فیلدهای اصلی شامل `id`, `book_id`, `book_title`, `volume_number`, `page_number`, `page_order`, `page_id`, `page_chunk_count`, `sub_order`, `child_order`, `content`, `lang`, `meta_info` و `footnotes` هستند.
