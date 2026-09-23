# ⏳ သင်ခန်းစာ (၁၁) - Queues, Jobs, Events နှင့် Mail Notifications
### (Lesson 11: Asynchronous Processing, Background Jobs, Queue Workers & Mail)

---

## 📌 မာတိကာ (Contents)
1. [Synchronous vs Asynchronous Processing ကွာခြားချက်](#၁-synchronous-vs-asynchronous-processing)
2. [Queue Drivers များနှင့် Database Queue Setup](#၂-queue-drivers-များနှင့်-database-queue-setup)
3. [Queue Job တစ်ခု ဖန်တီးခြင်းနှင့် Dispatch လုပ်ပုံ](#၃-queue-job-တစ်ခု-ဖန်တီးခြင်းနှင့်-dispatch-လုပ်ပုံ)
4. [Queue Workers Run ခြင်းနှင့် Supervisor စီမံခန့်ခွဲမှု](#၄-queue-workers-run-ခြင်းနှင့်-supervisor)
5. [Laravel Mailables ဖြင့် Email ပို့ဆောင်ခြင်း](#၅-laravel-mailables-ဖြင့်-email-ပို့ဆောင်ခြင်း)
6. [Events နှင့် Listeners စနစ် (Decoupled Architecture)](#၆-events-နှင့်-listeners-စနစ်)

---

## ၁။ Synchronous vs Asynchronous Processing

ပုံမှန်အားဖြင့် PHP သည် **Synchronous** (လိုင်းတစ်ခုပြီးမှ တစ်ခုလုပ်ဆောင်သည့်ပုံစံ) ဖြစ်သည်။

### ❌ ပုံမှန်ပြဿနာ (Synchronous Blocking):
အသုံးပြုသူသည် Register ခလုတ်နှိပ်လိုက်ချိန်တွင် Welcome Email ပို့ရန် ၅ စက္ကန့်ခန့် ကြာမြင့်ပါက Browser တွင် ၅ စက္ကန့်လုံးလုံး Loading လည်နေမည်ဖြစ်ပြီး User Experience အလွန်ဆိုးရွားစေသည်။

### ✅ Laravel Queue ၏ ဖြေရှင်းနည်း (Asynchronous Background Job):
Register လုပ်လိုက်သည်နှင့် အသုံးပြုသူထံ ချက်ချင်း Success ပြသလိုက်ပြီး Email ပို့သည့်တာဝန်ကို **Queue (တန်းစီဇယား)** ထဲသို့ ထည့်ပေးလိုက်သည်။ နောက်ကွယ်ရှိ **Queue Worker** က ၎င်းအလုပ်ကို Background တွင် ဆက်လက်လုပ်ဆောင်ပေးသည်။

```
[User Registration Request]
          │
          ├──► (1) Save User to Database
          ├──► (2) Push Job to Queue Table ──► Return "Success" to User immediately (< 100ms)
          │
    [Background Queue Worker]
          │ (Takes job from queue)
          ▼
    [Sends Welcome Email in Background] (5 seconds)
```

---

## ၂။ Queue Drivers များနှင့် Database Queue Setup

`config/queue.php` တွင် Queue Drivers များကို ရွေးချယ်နိုင်သည်:
* `sync` : ချက်ချင်း run သည် (Testing အတွက်သာ သုံးသည်)။
* `database` : Database Table ထဲတွင် Job များကို တန်းစီသိမ်းသည်။
* `redis` : မြန်နှုန်း အလွန်မြင့်မားသော In-memory Queue (Production စနစ်ကြီးများအတွက်)။

### Database Queue Setup ပြုလုပ်ခြင်း:
`.env` ဖိုင်တွင် Driver ပြောင်းပါ:
```ini
QUEUE_CONNECTION=database
```

Queue jobs များ သိမ်းဆည်းရန် Table ဆောက်ခြင်း:
```bash
php artisan queue:table
php artisan queue:failed-table
php artisan migrate
```

---

## ၃။ Queue Job တစ်ခု ဖန်တီးခြင်းနှင့် Dispatch လုပ်ပုံ

```bash
php artisan make:job SendWelcomeEmailJob
```

### (က) Job Class ရေးသားခြင်း (`app/Jobs/SendWelcomeEmailJob.php`):
`ShouldQueue` interface ပါဝင်ရပါမည်:

```php
namespace App\Jobs;

use App\Models\User;
use App\Mail\WelcomeMail;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    // Fail ဖြစ်ပါက အကြိမ်ရေ ၃ ကြိမ် ပြန်ကြိုးစားမည်
    public $tries = 3;

    public function __construct(public User $user)
    {
        //
    }

    public function handle(): void
    {
        // နောက်ကွယ်တွင် Email ပို့မည့် အလုပ်
        Mail::to($this->user->email)->send(new WelcomeMail($this->user));
    }
}
```

### (ခ) Controller မှ Job ကို ခေါ်ယူအသုံးပြုခြင်း (Dispatch):
```php
public function register(Request $request)
{
    $user = User::create([...]);

    // ၁။ ချက်ချင်း Queue ထဲသို့ ထည့်သွင်းခြင်း
    SendWelcomeEmailJob::dispatch($user);

    // ၂။ သတ်မှတ်မိနစ် အကြာမှ ပို့စေလိုပါက (Delay Dispatch)
    SendWelcomeEmailJob::dispatch($user)->delay(now()->addMinutes(5));

    return response()->json(['message' => 'အကောင့်ဖွင့်ခြင်း အောင်မြင်ပါသည်']);
}
```

---

## ၄။ Queue Workers Run ခြင်းနှင့် Supervisor

Queue ထဲ ရောက်နေသော အလုပ်များကို စတင် အလုပ်လုပ်စေရန် Terminal တွင် အောက်ပါ command ကို run ထားရသည်:

```bash
php artisan queue:work
```

> [!IMPORTANT]
> Live Production Server တွင် Terminal ကို အမြဲ ဖွင့်ထား၍မရသောကြောင့် Linux **Supervisor** ကို အသုံးပြုပြီး `queue:work` process သေမသွားစေရန် အလိုအလျောက် စောင့်ကြည့် run ပေးရသည်။

---

## ၅။ Laravel Mailables ဖြင့် Email ပို့ဆောင်ခြင်း

```bash
php artisan make:mail WelcomeMail
```

### Mailable Class (`app/Mail/WelcomeMail.php`):
```php
namespace App\Mail;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class WelcomeMail extends Mailable
{
    use Queueable, SerializesModels;

    public function __construct(public User $user) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Laravel Myanmar Community မှ ကြိုဆိုပါသည်',
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'emails.welcome', // resources/views/emails/welcome.blade.php
        );
    }
}
```

---

## ၆။ Events နှင့် Listeners စနစ်

စနစ်တစ်ခုတွင် Action တစ်ခု ဖြစ်ပွားသည့်အခါ (ဥပမာ- Order တင်လိုက်ခြင်း) နောက်ဆက်တွဲ အလုပ်များစွာ (Stock လျှော့ခြင်း၊ Email ပို့ခြင်း၊ SMS ပို့ခြင်း) ကို Controller ထဲ မရှုပ်ထွေးစေရန် **Event-Driven Architecture** ကို အသုံးပြုသည်။

```bash
php artisan make:event OrderPlaced
php artisan make:listener SendOrderInvoiceListener --event=OrderPlaced
```

```php
// Controller တွင် Event ထုတ်လွှင့်လိုက်ရုံသာ:
OrderPlaced::dispatch($order);

// Listener ထဲတွင် ShouldQueue ထည့်ထားပါက နောက်ဆက်တွဲ အလုပ်အားလုံး နောက်ကွယ်တွင် auto run သွားမည်
class SendOrderInvoiceListener implements ShouldQueue
{
    public function handle(OrderPlaced $event): void
    {
        // Send Invoice Logic
    }
}
```
