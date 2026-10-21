متن آماده ارسال، صرفاً بر اساس تست Chromium هدلس: Annotation 1

---

سلام و ممنون از پیگیری شما.
مشکل ما هنوز باقی مانده است و در هر سه کلاستر ایران تست کردم. در وضعیت فعلی صرف موفقیت DNS و TCP برای اطمینان از رفع مشکل کافی نیست.

برای بررسی دقیق‌تر، یک تست مستقل با **Chromium هدلس** روی لوکال و کانتینر سرور اجرا کردیم. آدرس شروع در هر دو اجرا یکسان بود:

```text
https://setadiran.ir/
```

این تست مستقل از منطق crawler است، هیچ اطلاعات ورود ارسال نمی‌کند و پس از تلاش اولیه، مرورگر را **۱۸۰ ثانیه باز نگه می‌دارد** تا بازشدن صفحات، پاپ‌آپ‌ها، تغییر آدرس و بسته‌شدن صفحات ثبت شود.

**کد تست قابل اجرا روی سرور**

نیازمند Playwright و Chromium نصب‌شده است؛ این وابستگی‌ها در ایمیج فعلی برنامه ما وجود دارند.

```bash
python3 -u - <<'PY'
import time
from datetime import datetime
from playwright.sync_api import sync_playwright

URL = "https://setadiran.ir/"
ALIVE_SECONDS = 180

def log(message):
    print(
        f"{datetime.now().isoformat(timespec='seconds')} {message}",
        flush=True,
    )

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    context = browser.new_context(ignore_https_errors=False)
    page_ids = {}

    def watch(page):
        number = len(page_ids) + 1
        page_ids[page] = number
        log(f"[page {number}] OPEN url={page.url}")

        page.on("close", lambda: log(f"[page {number}] CLOSED"))
        page.on("crash", lambda: log(f"[page {number}] CRASHED"))
        page.on("popup", lambda popup: log(
            f"[page {number}] POPUP created url={popup.url}"
        ))
        page.on("framenavigated", lambda frame: log(
            f"[page {number}] NAVIGATED {frame.url}"
        ) if frame == page.main_frame else None)
        page.on("domcontentloaded", lambda: log(
            f"[page {number}] DOM READY {page.url}"
        ))
        page.on("load", lambda: log(
            f"[page {number}] LOAD {page.url}"
        ))
        page.on("requestfailed", lambda request: log(
            f"[page {number}] REQUEST FAILED "
            f"{request.failure} {request.url}"
        ) if request.is_navigation_request() else None)
        page.on("response", lambda response: log(
            f"[page {number}] HTTP {response.status} {response.url}"
        ) if response.request.is_navigation_request() else None)

    context.on("page", watch)
    page = context.new_page()

    try:
        log(f"Opening {URL}")
        try:
            page.goto(URL, wait_until="commit", timeout=45000)
        except Exception as error:
            log(f"Initial navigation error: {error}")
            log("Keeping browser alive to observe other pages.")

        deadline = time.monotonic() + ALIVE_SECONDS
        next_summary = 0

        while time.monotonic() < deadline:
            pages = [
                item for item in context.pages
                if not item.is_closed()
            ]
            if not pages:
                log("No open pages remain.")
                break

            try:
                pages[0].wait_for_timeout(1000)
            except Exception:
                continue

            if time.monotonic() >= next_summary:
                for item in context.pages:
                    if not item.is_closed():
                        log(
                            f"[page {page_ids.get(item, '?')}] "
                            f"ALIVE {item.url}"
                        )
                next_summary = time.monotonic() + 10

    except KeyboardInterrupt:
        log("Stopped by operator.")
    finally:
        for item in context.pages:
            if item.is_closed():
                continue
            number = page_ids.get(item, "?")
            log(f"[page {number}] FINAL URL {item.url}")
            try:
                path = f"/tmp/setad-probe-page-{number}.png"
                item.screenshot(path=path, timeout=5000)
                log(f"[page {number}] SCREENSHOT {path}")
            except Exception as error:
                log(
                    f"[page {number}] Screenshot failed: "
                    f"{type(error).__name__}"
                )

        browser.close()
        log("Browser closed.")
PY
```

**خروجی واقعی روی لوکال**

```text
2026-10-21T16:59:24 [page 1] OPEN url=about:blank
2026-10-21T16:59:24 Opening https://setadiran.ir/
2026-10-21T16:59:24 [page 1] HTTP 301 https://setadiran.ir/
2026-10-21T16:59:24 [page 1] HTTP 200 https://setadiran.ir/setad/cms
2026-10-21T16:59:24 [page 1] NAVIGATED https://setadiran.ir/setad/cms
2026-10-21T16:59:25 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T16:59:26 [page 1] HTTP 200 https://sso2.setadiran.ir/portal/local-storage-check
2026-10-21T16:59:26 [page 1] DOM READY https://setadiran.ir/setad/cms
2026-10-21T16:59:31 [page 1] LOAD https://setadiran.ir/setad/cms
2026-10-21T16:59:36 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T16:59:46 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T16:59:56 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:00:06 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:00:16 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:00:26 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:00:36 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:00:46 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:00:56 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:01:07 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:01:17 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:01:27 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:01:37 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:01:47 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:01:57 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:02:07 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:02:17 [page 1] ALIVE https://setadiran.ir/setad/cms
2026-10-21T17:02:25 [page 1] FINAL URL https://setadiran.ir/setad/cms
2026-10-21T17:02:26 [page 1] SCREENSHOT /tmp/setad-probe-page-1.png
2026-10-21T17:02:26 [page 1] CLOSED
2026-10-21T17:02:26 Browser closed.
```

**خروجی ثبت‌شده روی سرور برای همان آدرس شروع**

```text
2026-10-21T13:25:36 [page 1] OPEN url=about:blank
2026-10-21T13:25:36 Opening https://setadiran.ir/
2026-10-21T13:25:36 [page 1] REQUEST FAILED net::ERR_EMPTY_RESPONSE https://setadiran.ir/
2026-10-21T13:25:36 Initial navigation error: Page.goto: net::ERR_EMPTY_RESPONSE at https://setadiran.ir/
Call log:
  - navigating to "https://setadiran.ir/", waiting until "commit"

2026-10-21T13:25:36 Keeping browser alive to observe other pages.
2026-10-21T13:25:36 [page 1] NAVIGATED chrome-error://chromewebdata/
2026-10-21T13:25:36 [page 1] DOM READY chrome-error://chromewebdata/
2026-10-21T13:25:36 [page 1] LOAD chrome-error://chromewebdata/
2026-10-21T13:25:37 [page 1] ALIVE chrome-error://chromewebdata/
2026-10-21T13:25:47 [page 1] ALIVE chrome-error://chromewebdata/
2026-10-21T13:25:57 [page 1] ALIVE chrome-error://chromewebdata/
```

خروجی سرور که اینجا نقل شده تا همین زمان در دسترس است؛ زمان‌های دو محیط نیز عین خروجی ثبت‌شده هستند و به منطقه زمانی یکسان تبدیل نشده‌اند.

**تفاوت مشاهده‌شده**

روی لوکال، مرورگر پاسخ `301` دریافت می‌کند، سپس صفحه `/setad/cms` با وضعیت `200` باز می‌شود و درخواست مرتبط با `sso2` هم پاسخ `200` می‌گیرد.

روی سرور، همان آدرس اولیه بدون دریافت پاسخ HTTP با `ERR_EMPTY_RESPONSE` شکست می‌خورد. رویدادهای `DOM READY` و `LOAD` بعدی مربوط به **صفحه خطای داخلی Chromium** هستند، نه سایت ستاد.

لطفاً همین تست را از کلاستر `c15`، با آدرس شروع **`https://setadiran.ir/`** اجرا کنید و خروجی مرورگر را ارسال بفرمایید تا پیش از انتقال اپ و دیتابیس، موفقیت دریافت HTTP و بارگذاری صفحه در آن کلاستر مشخص شود.

با تشکر.
