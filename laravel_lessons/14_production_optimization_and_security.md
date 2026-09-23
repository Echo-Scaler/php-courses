# 🚀 သင်ခန်းစာ (၁၄) - Production Deployment, Performance Optimization နှင့် Security
### (Lesson 14: Going Live, Caching Strategies, Security Hardening & Maintenance)

---

## 📌 မာတိကာ (Contents)
1. [Production Deployment ကြိုတင်ပြင်ဆင်ခြင်း Checklist](#၁-production-deployment-ကြိုတင်ပြင်ဆင်ခြင်း-checklist)
2. [မဖြစ်မနေ Run ရမည့် Laravel Performance Optimization Commands](#၂-မဖြစ်မနေ-run-ရမည့်-laravel-performance-optimization-commands)
3. [Environment Hardening နှင့် လုံခြုံရေး သတိပြုဖွယ်ရာများ](#၃-environment-hardening-နှင့်-လုံခြုံရေး)
4. [Rate Limiting စနစ်ဖြင့် Brute Force နှင့် DDoS တားဆီးခြင်း](#၄-rate-limiting-စနစ်)
5. [Production Caching (Redis ဖြင့် မြန်နှုန်းမြှင့်တင်ခြင်း)](#၅-production-caching)
6. [Logging Channels နှင့် Error Monitoring](#၆-logging-channels-နှင့်-error-monitoring)

---

## ၁။ Production Deployment ကြိုတင်ပြင်ဆင်ခြင်း Checklist

Application တစ်ခုကို စမ်းသပ်သည့် Local Server မှ အင်တာနက်ပေါ်ရှိ Live Production Server သို့ တင်သည့်အခါ အောက်ပါ အဆင့် ၅ ဆင့်ကို မဖြစ်မနေ စစ်ဆေးရပါသည်:

* [ ] `.env` ဖိုင်တွင် `APP_ENV=production` နှင့် `APP_DEBUG=false` ထားရှိခြင်း။
* [ ] Production Dependencies သာ Install ပြုလုပ်ခြင်း (`composer install --no-dev -o`)။
* [ ] Nginx/Apache Web Server ၏ Document Root ကို `public/` directory အဖြစ်သာ ညွှန်ပြထားခြင်း။
* [ ] `storage/` နှင့် `bootstrap/cache/` directories များကို Web Server (e.g. `www-data`) မှ Write Access ပေးထားခြင်း (`chmod -R 775 storage bootstrap/cache`)။
* [ ] `php artisan storage:link` ချိတ်ဆက်ထားခြင်း။

---

## ၂။ မဖြစ်မနေ Run ရမည့် Laravel Performance Optimization Commands

Laravel အက်ပလီကေးရှင်းတစ်ခုသည် စတင် Run တိုင်း Config ဖိုင်များ၊ Routes ဖိုင်များနှင့် Blade Templates များကို ဖတ်ယူရသဖြင့် Production တွင် အောက်ပါ Commands များကို Run ပေးခြင်းဖြင့် Performance ကို ၃ ဆ မှ ၅ ဆ အထိ မြန်ဆန်စေသည်:

```bash
# ၁။ Configuration ဖိုင်များကို ဖိုင်တစ်ခုတည်းအဖြစ် စုစည်း Cache လုပ်ခြင်း
php artisan config:cache

# ၂။ Routes များကို Cache ပြုလုပ်ခြင်း (URL စစ်ဆေးမှု အလွန်မြန်ဆန်သွားမည်)
php artisan route:cache

# ③။ Blade Templates များကို Pre-compile ပြုလုပ်ခြင်း
php artisan view:cache

# ၄။ Event Listeners များကို Cache ပြုလုပ်ခြင်း
php artisan event:cache

# ၅။ အထက်ပါ Cache အားလုံးကို တစ်ပြိုင်နက် လုပ်ဆောင်ပေးသော အထူး Command
php artisan optimize
```

> [!CAUTION]
> အကယ်၍ Production ပေါ်တွင် Code အသစ်များ Update ပြုလုပ်ပြီးနောက် ပြင်ဆင်ချက်များ မပေါ်လာပါက Cache များကို ရှင်းလင်းပေးရပါသည်:
> ```bash
> php artisan optimize:clear
> ```

---

## ၃။ Environment Hardening နှင့် လုံခြုံရေး

### (က) `APP_DEBUG=false` အလွန်အရေးကြီးပုံ
အကယ်၍ Production Server ပေါ်တွင် `APP_DEBUG=true` ဖြစ်နေပါက Database Error သို့မဟုတ် Exception တစ်ခုခု တက်သည့်အခါ Laravel ၏ Ignition Error Screen တွင် **Database Password, Secret API Keys, .env တန်ဖိုးများအားလုံး အင်တာနက်ပေါ်တွင် လူတိုင်း မြင်တွေ့သွားနိုင်ပါသည်**။

```ini
APP_NAME="Myanmar Production System"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://myapp.com
```

### (ခ) HTTPS သို့ အတင်းအကျပ် ပို့ဆောင်ခြင်း (Force HTTPS)
`app/Providers/AppServiceProvider.php` တွင်:
```php
use Illuminate\Support\Facades\URL;

public function boot(): void
{
    if (app()->environment('production')) {
        URL::forceScheme('https');
    }
}
```

---

## ၄။ Rate Limiting စနစ်

Hacker များ Login Password ကို စက်ဖြင့် တောက်လျှောက် ရိုက်စစ်ခြင်း (Brute Force Attack) နှင့် API ကို ဒုက္ခပေးခြင်း (DDoS) မှ ကာကွယ်ရန် **Rate Limiter** ကို အသုံးပြုသည်:

```php
// bootstrap/app.php သို့မဟုတ် AppServiceProvider
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Http\Request;

RateLimiter::for('api', function (Request $request) {
    // တစ်မိနစ်လျှင် အများဆုံး Request ၆၀ သာ ခွင့်ပြုမည်
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

RateLimiter::for('login', function (Request $request) {
    // Login စာမျက်နှာကို ၁ မိနစ်လျှင် ၅ ကြိမ်သာ စမ်းသပ်ခွင့်ပေးမည်
    return Limit::perMinute(5)->by($request->ip());
});
```

---

## ၅။ Production Caching

Database ပေါ်သို့ ဝန်မပိစေရန် မကြာခဏ အပြောင်းအလဲမရှိသော Data များကို Cache တွင် သိမ်းဆည်းနိုင်သည်:

```php
use Illuminate\Support\Facades\Cache;

// ပစ္စည်း အုပ်စုများကို Cache ထဲတွင် ၁ နာရီကြာ သိမ်းထားမည်
$categories = Cache::remember('active_categories', 3600, function () {
    return Category::where('is_active', true)->get();
});

// Cache ရှင်းလင်းခြင်း
Cache::forget('active_categories');
```

---

## ၆။ Logging Channels နှင့် Error Monitoring

Error များကို နေ့စဉ် ဖိုင်ခွဲ၍ သိမ်းဆည်းရန် `config/logging.php` တွင် `daily` channel ကို အသုံးပြုသင့်သည်:

```ini
# .env ဖိုင်တွင်
LOG_CHANNEL=daily
LOG_LEVEL=error
```

```php
use Illuminate\Support\Facades\Log;

try {
    // Critical Action
} catch (\Exception $e) {
    // storage/logs/laravel-YYYY-MM-DD.log ထဲသို့ အသေးစိတ် မှတ်တမ်းဝင်မည်
    Log::error('ငွေပေးချေမှု မအောင်မြင်ပါ: ' . $e->getMessage(), [
        'user_id' => auth()->id(),
        'trace'   => $e->getTraceAsString(),
    ]);
}
```
---

## 🎓 သင်တန်း ပြီးမြောက်ခြင်း အထိမ်းအမှတ် (Congratulations!)

အထက်ပါ သင်ခန်းစာ (၁၄) ခုလုံးကို အဆင့်ဆင့် စနစ်တကျ လေ့လာပြီး လက်တွေ့ ကုဒ်များ ရေးသားလိုက်နာပါက သင်သည် ခေတ်မီဆန်းသစ်ပြီး လုံခြုံစိတ်ချရသော Enterprise Laravel Web Applications များနှင့် RESTful APIs များကို ကျွမ်းကျင်ပိုင်နိုင်စွာ တည်ဆောက်နိုင်သော **Professional Laravel Developer** တစ်ဦး ဖြစ်လာပြီ ဖြစ်ပါသည်။
