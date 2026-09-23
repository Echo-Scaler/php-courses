# 🔐 သင်ခန်းစာ (၈) - Middleware နှင့် Authentication / Authorization (လုံခြုံရေးစနစ်)
### (Lesson 8: Custom Middlewares, Authentication Systems, Gates & Policies)

---

## 📌 မာတိကာ (Contents)
1. [Middleware ဆိုတာဘာလဲ? (Onion Architecture အလုပ်လုပ်ပုံ)](#၁-middleware-ဆိုတာဘာလဲ)
2. [Custom Middleware ဖန်တီးခြင်းနှင့် Register ပြုလုပ်ခြင်း](#၂-custom-middleware-ဖန်တီးခြင်းနှင့်-register-ပြုလုပ်ခြင်း)
3. [Middleware သို့ Parameters ပေးပို့ခြင်း](#၃-middleware-သို့-parameters-ပေးပို့ခြင်း)
4. [Session-based Authentication Flow (Manual Login & Logout)](#၄-session-based-authentication-flow)
5. [Laravel Breeze Starter Kit မိတ်ဆက်](#၅-laravel-breeze-starter-kit-မိတ်ဆက်)
6. [Authorization: Gates နှင့် Policies ကွာခြားချက်နှင့် အသုံးချပုံ](#၆-authorization-gates-နှင့်-policies)

---

## ၁။ Middleware ဆိုတာဘာလဲ?

**Middleware** ဆိုသည်မှာ HTTP Request တစ်ခုသည် Controller သို့ မရောက်ရှိမီ (သို့မဟုတ်) Response သည် Browser ဆီသို့ မထွက်ခွာမီ **ကြားဖြတ် စစ်ဆေးပေးသော လုံခြုံရေးတံခါးပေါက် (Filter/Pipeline)** ဖြစ်သည်။

```
[HTTP Request]
       │
       ▼
┌───────────────────────────────┐
│ 1. CheckMaintenanceMiddleware  │ ◄── Website ပိတ်ထားသလား?
└──────────────┬────────────────┘
               │ (Next)
               ▼
┌───────────────────────────────┐
│ 2. Authenticate Middleware    │ ◄── Login ဝင်ထားသလား?
└──────────────┬────────────────┘
               │ (Next)
               ▼
┌───────────────────────────────┐
│ 3. EnsureUserIsAdmin          │ ◄── Role သည် Admin ဟုတ်ပါသလား?
└──────────────┬────────────────┘
               │ (Pass)
               ▼
       [Controller Action]
```

---

## ၂။ Custom Middleware ဖန်တီးခြင်းနှင့် Register ပြုလုပ်ခြင်း

ဥပမာ - User ၏ Role သည် Admin မဟုတ်ပါက ဝင်ရောက်ခွင့် မပေးမည့် Middleware တစ်ခု တည်ဆောက်ခြင်း:

```bash
php artisan make:middleware EnsureUserIsAdmin
```

### (က) Middleware ကုဒ် (`app/Http/Middleware/EnsureUserIsAdmin.php`):
```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        // အကယ်၍ User Login မဝင်ထားသေးပါက သို့မဟုတ် role သည် admin မဟုတ်ပါက
        if (! $request->user() || $request->user()->role !== 'admin') {
            abort(403, 'ဤစာမျက်နှာကို Admin များသာ ဝင်ရောက်ခွင့် ရှိပါသည်။');
        }

        // အားလုံး ကိုက်ညီပါက နောက်အဆင့်သို့ ဆက်သွားခွင့်ပြုသည်
        return $next($request);
    }
}
```

### (ခ) Laravel 11 တွင် Middleware ကို Alias သတ်မှတ်ခြင်း (`bootstrap/app.php`):
```php
use App\Http\Middleware\EnsureUserIsAdmin;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Alias သတ်မှတ်ခြင်း
        $middleware->alias([
            'admin' => EnsureUserIsAdmin::class,
        ]);
    })
    ->create();
```

### (ဂ) Route တွင် ချိတ်ဆက်အသုံးပြုပုံ:
```php
Route::middleware(['auth', 'admin'])->prefix('admin')->group(function () {
    Route::get('/dashboard', [AdminDashboardController::class, 'index']);
});
```

---

## ၃။ Middleware သို့ Parameters ပေးပို့ခြင်း

Role အမျိုးမျိုး (ဥပမာ- admin, superadmin, manager) ကို စစ်ဆေးနိုင်သော Dynamic Middleware:

```php
public function handle(Request $request, Closure $next, string ...$roles): Response
{
    if (! in_array($request->user()?->role, $roles)) {
        abort(403, 'Unauthorized access.');
    }

    return $next($request);
}

// routes/web.php တွင် သုံးစွဲပုံ:
Route::get('/analytics', [AnalyticsController::class, 'index'])->middleware('role:admin,manager');
```

---

## ၄။ Session-based Authentication Flow

Laravel တွင် Manual Login/Logout စနစ်ကို အလွန်လွယ်ကူစွာ ရေးသားနိုင်ပါသည်:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Http\Request;

class AuthController extends Controller
{
    // Login ဝင်ရောက်ခြင်း
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email'    => ['required', 'email'],
            'password' => ['required'],
        ]);

        $remember = $request->boolean('remember');

        // Auth::attempt က Database ထဲမှ password hash ကို auto စစ်ဆေးပေးသည်
        if (Auth::attempt($credentials, $remember)) {
            $request->session()->regenerate(); // Session Fixation Attack ကာကွယ်ခြင်း

            return redirect()->intended('dashboard'); // မူလသွားချင်ခဲ့သော စာမျက်နှာသို့ ပို့ပေးသည်
        }

        return back()->withErrors([
            'email' => 'Email သို့မဟုတ် စကားဝှက် မှားယွင်းနေပါသည်။',
        ])->onlyInput('email');
    }

    // Logout ထွက်ခြင်း
    public function logout(Request $request)
    {
        Auth::logout();

        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return redirect('/');
    }
}
```

---

## ၅။ Laravel Breeze Starter Kit မိတ်ဆက်

လက်တွေ့ လုပ်ငန်းခွင်တွင် Login, Registration, Password Reset, Email Verification စသည်တို့ကို အစမှ လက်ဖြင့်ရေးမည့်အစား Laravel ၏ တရားဝင် **Breeze Starter Kit** ကို အသုံးပြုကြသည်။

```bash
# Composer ဖြင့် Breeze ထည့်သွင်းခြင်း
composer require laravel/breeze --dev

# Blade + Tailwind CSS ဖြင့် Auth Scaffold ဆောက်ခြင်း
php artisan breeze:install blade

# Database migrate လုပ်ပြီး Assets build လုပ်ခြင်း
php artisan migrate
npm install && npm run dev
```

---

## ၆။ Authorization: Gates နှင့် Policies

* **Authentication (Who are you?)**: အသုံးပြုသူ မည်သူဖြစ်ကြောင်း စစ်ခြင်း (Login)။
* **Authorization (What can you do?)**: အသုံးပြုသူတွင် ဤအရာကို လုပ်ပိုင်ခွင့်ရှိမရှိ စစ်ခြင်း (Permissions)။

### (က) Gates (ရိုးရှင်းသော ခွင့်ပြုချက်များ)
`app/Providers/AppServiceProvider.php` တွင် သတ်မှတ်သည်:
```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::define('access-admin-panel', function (User $user) {
        return $user->role === 'admin';
    });
}
```

### (ခ) Policies (Model တစ်ခုချင်းစီအတွက် ခွင့်ပြုချက် စည်းမျဉ်းများ)
```bash
php artisan make:policy PostPolicy --model=Post
```

```php
// app/Policies/PostPolicy.php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    // Post ရေးတင်ခဲ့သူ ကိုယ်တိုင်သာ Update ပြုလုပ်ခွင့် ပေးမည်
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

#### သုံးစွဲပုံ (Controller & Blade):
```php
// Controller တွင်:
public function edit(Post $post)
{
    $this->authorize('update', $post); // မကိုက်ညီပါက 403 Forbidden ပစ်ပေးသည်
    return view('posts.edit', compact('post'));
}
```

```html
<!-- Blade Template တွင်: -->
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}" class="btn btn-warning">Edit</a>
@endcan
```
