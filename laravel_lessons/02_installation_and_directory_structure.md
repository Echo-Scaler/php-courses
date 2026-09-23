# 📁 သင်ခန်းစာ (၂) - Laravel Project တည်ဆောက်ခြင်းနှင့် Directory Structure အသေးစိတ်
### (Lesson 2: Project Installation, Setup & Deep Dive into Directory Structure)

---

## 📌 မာတိကာ (Contents)
1. [စနစ် လိုအပ်ချက်များနှင့် စတင် Install လုပ်ခြင်း](#၁-စနစ်-လိုအပ်ချက်များနှင့်-စတင်-install-လုပ်ခြင်း)
2. [Artisan CLI နှင့် ပထမဆုံး Local Server စတင် Run ခြင်း](#၂-artisan-cli-နှင့်-ပထမဆုံး-local-server-စတင်-run-ခြင်း)
3. [Laravel Directory Structure အပြည့်အစုံ ရှင်းလင်းချက်](#၃-laravel-directory-structure-အပြည့်အစုံ-ရှင်းလင်းချက်)
4. [Environment Configuration (`.env`) ဖိုင်၏ အရေးကြီးပုံ](#၄-environment-configuration-env-ဖိုင်၏-အရေးကြီးပုံ)
5. [အသုံးအများဆုံး Artisan Commands စာရင်း](#၅-အသုံးအများဆုံး-artisan-commands-စာရင်း)
6. [လက်တွေ့လေ့ကျင့်ရန် Lab Exercise](#၆-လက်တွေ့လေ့ကျင့်ရန်-lab-exercise)

---

## ၁။ စနစ် လိုအပ်ချက်များနှင့် စတင် Install လုပ်ခြင်း

Laravel 11+ ကို စတင်အသုံးပြုရန် သင့်စက်တွင် အောက်ပါ Software များ ရှိထားရန် လိုအပ်ပါသည်:
* **PHP >= 8.2** (BCMath, Ctype, Fileinfo, JSON, Mbstring, OpenSSL, PDO, Tokenizer, XML extension များ ဖွင့်ထားရမည်)
* **Composer** (PHP ၏ Package Dependency Manager)
* **Database Engine** (MySQL 8.0+, MariaDB 10.3+, PostgreSQL 12+, သို့မဟုတ် SQLite 3.35+)
* **Node.js & NPM** (Frontend Asset Bundling - Vite အတွက်)

### (က) Composer ဖြင့် Project အသစ် ဆောက်ခြင်း
Terminal သို့မဟုတ် Command Prompt တွင် အောက်ပါ command ကို ရိုက်ထည့်ပါ:

```bash
# syntax: composer create-project laravel/laravel <project-name>
composer create-project laravel/laravel my-first-laravel
```

### (ခ) Project Folder ထဲသို့ ဝင်ရောက်ခြင်း
```bash
cd my-first-laravel
```

---

## ၂။ Artisan CLI နှင့် ပထမဆုံး Local Server စတင် Run ခြင်း

**Artisan** ဆိုသည်မှာ Laravel တွင် Built-in ပါဝင်သော Command-Line Interface (CLI) ဖြစ်ပြီး Boilerplate ကုဒ်များ ဆောက်ခြင်း၊ Database စီမံခြင်းနှင့် Server run ခြင်းတို့ကို အလွန်မြန်ဆန်စွာ လုပ်ဆောင်ပေးသည်။

```bash
# Built-in Development Web Server စတင်ခြင်း
php artisan serve
```

အောက်ပါအတိုင်း ပေါ်လာပါမည်:
```
INFO  Server running on [http://127.0.0.1:8000].
Press Ctrl+C to stop the server
```
Browser တွင် `http://127.0.0.1:8000` သို့ သွားရောက်ကြည့်ပါက Laravel Welcome Page ကို စတင်တွေ့မြင်ရမည်ဖြစ်ပါသည်။

---

## ၃။ Laravel Directory Structure အပြည့်အစုံ ရှင်းလင်းချက်

Laravel ၏ Project Folder ဖွဲ့စည်းပုံသည် MVC စနစ်အရ အလွန်စနစ်ကျပါသည်။

```
my-first-laravel/
├── app/                  # Application Core Logic (Models, Controllers, Middleware)
├── bootstrap/            # Framework စတင် run ရန် Bootstrap ဖိုင်များ (app.php)
├── config/               # App configuration ဖိုင်များ (database.php, mail.php, etc.)
├── database/             # Migrations, Model Factories, Seeders များ
├── public/               # Web Server ၏ Document Root (index.php, CSS, JS, Images)
├── resources/            # Views (Blade templates), Raw CSS/JS (Vite assets)
├── routes/               # URL Routing သတ်မှတ်ချက်များ (web.php, api.php, console.php)
├── storage/              # Logs, Compiled templates, User uploaded files
├── tests/                # Feature & Unit Testing ဖိုင်များ
├── vendor/               # Composer dependencies များ
├── .env                  # Environment Variables လျှို့ဝှက် config ဖိုင်
├── composer.json         # PHP Packages စာရင်း
├── package.json          # Node/NPM Packages စာရင်း (Vite, Tailwind, etc.)
└── vite.config.js        # Frontend bundler configuration
```

---

### Folder တစ်ခုချင်းစီ၏ အသေးစိတ် တာဝန်များ:

#### ၁။ `app/` Folder (အဓိက ဦးနှောက်)
* `app/Http/Controllers/` : အသုံးပြုသူ၏ Request ကို လက်ခံပြီး Model/View သို့ ပို့ပေးသည့် Controllers များ။
* `app/Models/` : Database Table တစ်ခုချင်းစီကို ကိုယ်စားပြုသော Eloquent Model Classes များ (ဥပမာ- `User.php`, `Product.php`)။
* `app/Http/Middleware/` : Request ကို ကြားဖြတ်စစ်ဆေးပေးသည့် Logic များ (ဥပမာ- Admin ဟုတ်မဟုတ် စစ်ဆေးခြင်း)။
* `app/Providers/` : Service Providers များ (AppServiceProvider, EventServiceProvider)။

#### ၂။ `config/` Folder (စနစ် စီမံခန့်ခွဲမှု)
Database settings, Authentication guards, Mail servers, Caching drivers စသည့် Application Configuration အားလုံးကို ဤနေရာတွင် စီမံသည်။

#### ၃။ `database/` Folder (ဒေတာဘေ့စ် စီမံခန့်ခွဲမှု)
* `database/migrations/` : Database Table များကို ကုဒ်ဖြင့် ဖန်တီးသော Version Control ဖိုင်များ။
* `database/seeders/` : စမ်းသပ်ရန် သို့မဟုတ် Master Data ထည့်သွင်းပေးသောဖိုင်များ။
* `database/factories/` : Faker ဒေတာများ အလိုအလျောက် ထုတ်လုပ်ပေးသော ဖိုင်များ။

#### ၄။ `public/` Folder (အပြင်လောကနှင့် ထိတွေ့ရာနေရာ)
* `index.php` သည် Web Traffic အားလုံး၏ ဝင်ပေါက်ဖြစ်သည်။
* Compiled CSS, Javascript, ပုံရိပ်များနှင့် Static Assets များကို ဤနေရာတွင်သာ ထားရှိရသည်။

#### ၅။ `resources/` Folder (အသုံးပြုသူ မြင်ကွင်း)
* `resources/views/` : Blade Template HTML ဖိုင်များ (`.blade.php`) ထားရှိရာ နေရာ။
* `resources/css/` နှင့် `resources/js/` : Vite ဖြင့် bundle လုပ်မည့် Frontend ကုဒ်များ။

#### ၆။ `routes/` Folder (လမ်းကြောင်းများ)
* `routes/web.php` : Browser ပေါ်တွင် ပြသမည့် စာမျက်နှာ Routes များ (Session, CSRF ပါဝင်သည်)။
* `routes/api.php` : Mobile Apps သို့မဟုတ် Frontend Frameworks များအတွက် Stateless REST API Routes များ။
* `routes/console.php` : Custom Artisan CLI Commands များ ရေးသားရာနေရာ။

#### ၇။ `storage/` Folder (ယာယီနှင့် ဒေတာသိုလှောင်ရာ)
* `storage/logs/laravel.log` : Error ဖြစ်တိုင်း မှတ်တမ်းဝင်သည့် Log ဖိုင်။
* `storage/app/public/` : အသုံးပြုသူ တင်လိုက်သော ပုံများ၊ PDF ဖိုင်များ သိမ်းဆည်းရာနေရာ။

---

## ၄။ Environment Configuration (`.env`) ဖိုင်၏ အရေးကြီးပုံ

`.env` ဖိုင်သည် **Server တစ်ခုချင်းစီ၏ လျှို့ဝှက် Settings များကို သိမ်းဆည်းရာဖိုင်** ဖြစ်ပါသည်။
*(သတိပြုရန်: ဤဖိုင်ကို Git repository ထဲသို့ commit လုံးဝ မထည့်ရပါ)*

```ini
# Application အမည်နှင့် Mode
APP_NAME="Myanmar E-Commerce"
APP_ENV=local
APP_KEY=base64:3k7zW... # အရေးကြီးသော Encryption Key
APP_DEBUG=true           # Local တွင်သာ true ထားရမည်၊ Live Server တွင် အမြဲ false ထားပါ
APP_URL=http://127.0.0.1:8000

# Database ချိတ်ဆက်မှု Setting
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=my_laravel_db
DB_USERNAME=root
DB_PASSWORD=secret

# Session & Cache Driver
SESSION_DRIVER=database
QUEUE_CONNECTION=database
CACHE_STORE=database
```

> [!IMPORTANT]
> ပရောဂျက်ကို clone ချပြီးစတွင် `.env.example` ကို `.env` အဖြစ် ကူးယူပြီး အောက်ပါ command ဖြင့် `APP_KEY` ကို အသစ်ထုတ်ပေးရပါသည်:
> ```bash
> cp .env.example .env
> php artisan key:generate
> ```

---

## ၅။ အသုံးအများဆုံး Artisan Commands စာရင်း

| Command | လုပ်ဆောင်ချက် |
| :--- | :--- |
| `php artisan serve` | Development Local Web Server run ခြင်း |
| `php artisan route:list` | Application တွင် ရှိသမျှ URL Routes များကို ဇယားဖြင့် ပြသခြင်း |
| `php artisan make:controller [Name]` | Controller အသစ် ဆောက်ခြင်း |
| `php artisan make:model [Name] -m` | Model နှင့်အတူ Migration ပါ တစ်ပြိုင်နက် ဆောက်ခြင်း |
| `php artisan migrate` | ရေးဆွဲထားသော Database Migrations များကို run ခြင်း |
| `php artisan make:request [Name]` | Form Validation Request class အသစ် ဆောက်ခြင်း |
| `php artisan tinker` | Terminal ပေါ်တွင် Laravel ကုဒ်များကို Interactive စမ်းသပ် run ခြင်း |

---

## ၆။ လက်တွေ့လေ့ကျင့်ရန် Lab Exercise

1. သင့်ကွန်ပျူတာတွင် `composer create-project laravel/laravel my-blog` ဖြင့် Project တစ်ခု ဆောက်ပါ။
2. `.env` ဖိုင်ထဲတွင် `APP_NAME="My Learning Blog"` ဟု ပြင်ဆင်ပါ။
3. `php artisan serve` run ပြီး Browser တွင် `http://127.0.0.1:8000` ဖြင့် ဝင်ရောက်ကြည့်ရှုပါ။
4. Terminal တွင် `php artisan --version` နှင့် `php artisan route:list` များကို ရိုက်နှိပ် စမ်းသပ်ကြည့်ပါ။
