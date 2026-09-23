# 🧭 သင်ခန်းစာ (၃) - Routing နှင့် Controller စနစ် (HTTP Request စီမံခန့်ခွဲမှု)
### (Lesson 3: Routing Architecture, HTTP Verbs, Controllers & Route Model Binding)

---

## 📌 မာတိကာ (Contents)
1. [Routing ဆိုတာဘာလဲနှင့် HTTP Methods များ](#၁-routing-ဆိုတာဘာလဲနှင့်-http-methods-များ)
2. [Route Parameters နှင့် Regex Constraints](#၂-route-parameters-နှင့်-regex-constraints)
3. [Named Routes နှင့် `route()` Helper အသုံးချပုံ](#၃-named-routes-နှင့်-route-helper-အသုံးချပုံ)
4. [Route Groups (Prefix, Names, Middlewares)](#၄-route-groups-prefix-names-middlewares)
5. [Controllers အမျိုးအစားများ (Basic, Resource, Single Action)](#၅-controllers-အမျိုးအစားများ)
6. [Route Model Binding (Implicit vs Explicit)](#၆-route-model-binding)
7. [လုပ်ငန်းခွင်သုံး CRUD Controller ကုဒ်နမူနာ အပြည့်အစုံ](#၇-လုပ်ငန်းခွင်သုံး-crud-controller-ကုဒ်နမူနာ)

---

## ၁။ Routing ဆိုတာဘာလဲနှင့် HTTP Methods များ

**Routing** ဆိုသည်မှာ အသုံးပြုသူ Browser မှ ရိုက်ထည့်လိုက်သော URL Path (ဥပမာ- `/products`, `/login`) ကို လက်ခံပြီး မည်သည့် Controller သို့မဟုတ် Logic ထံသို့ ပို့ဆောင်ပေးရမည်ကို လမ်းညွှန်ပေးသော စနစ်ဖြစ်ပါသည်။

Web Routes များကို `routes/web.php` ဖိုင်တွင် ရေးသားရသည်။

### အဓိက HTTP Methods (Verbs) များ:
```php
use Illuminate\Support\Facades\Route;

// ၁။ ဒေတာများကို ဖတ်ရှုရယူရန် (Read)
Route::get('/users', function () {
    return 'User list';
});

// ၂။ ဒေတာ အသစ် ဖန်တီးသိမ်းဆည်းရန် (Create)
Route::post('/users', function () {
    return 'User created';
});

// ၃။ ရှိပြီးသား ဒေတာကို အပြည့်အစုံ/တစိတ်တပိုင်း ပြင်ဆင်ရန် (Update)
Route::put('/users/{id}', function ($id) {
    return 'User fully updated';
});
Route::patch('/users/{id}', function ($id) {
    return 'User partially updated';
});

// ၄။ ဒေတာကို ဖျက်ပစ်ရန် (Delete)
Route::delete('/users/{id}', function ($id) {
    return 'User deleted';
});
```

---

## ၂။ Route Parameters နှင့် Regex Constraints

### (က) Required Parameter (မဖြစ်မနေ ထည့်ရမည့် တန်ဖိုး)
```php
Route::get('/users/{id}', function (string $id) {
    return "User ID: " . $id;
});
```

### (ခ) Optional Parameter (မပါလဲ ရသည်)
URL တွင် parameter မပါလာပါက default တန်ဖိုး သတ်မှတ်ပေးရသည်:
```php
Route::get('/posts/{slug?}', function (string $slug = 'all') {
    return "Showing posts for category: " . $slug;
});
```

### (ဂ) Regular Expression Constraints (URL တန်ဖိုး စစ်ဆေးခြင်း)
မလိုလားအပ်သော URL အမှားများကို ကာကွယ်ရန် `where` အသုံးပြုနိုင်သည်:
```php
// ID သည် ဂဏန်းသီးသန့်သာ ဖြစ်ရမည်
Route::get('/orders/{id}', function (string $id) {
    return "Order #{$id}";
})->whereNumber('id');

// Name သည် အက္ခရာ သီးသန့်သာ ဖြစ်ရမည်
Route::get('/user/{name}', function (string $name) {
    return "Username: {$name}";
})->whereAlpha('name');
```

---

## ၃။ Named Routes နှင့် `route()` Helper အသုံးချပုံ

Named Routes သည် Route တစ်ခုချင်းစီကို သီးသန့် အမည်တစ်ခု ပေးထားခြင်း ဖြစ်သည်။ နောင်တစ်ချိန်တွင် URL path ပြောင်းလဲသွားသော်လည်း ကုဒ်များ လိုက်ပြင်စရာ မလိုတော့ဘဲ အလိုအလျောက် အဆင်ပြေစေသည်။

```php
// Route ကို အမည်ပေးခြင်း
Route::get('/user/profile/settings', [ProfileController::class, 'edit'])->name('profile.settings');

// Controller သို့မဟုတ် View ထဲတွင် ပြန်ခေါ်သုံးပုံ:
// 1. Redirection:
return redirect()->route('profile.settings');

// 2. Blade Link ထဲတွင်:
// <a href="{{ route('profile.settings') }}">Settings</a>

// 3. Parameters ပါဝင်သော Named Route:
Route::get('/products/{id}', [ProductController::class, 'show'])->name('products.show');
// သုံးစွဲပုံ: route('products.show', ['id' => 45]) -> URL: /products/45
```

---

## ၄။ Route Groups (Prefix, Names, Middlewares)

လုပ်ငန်းခွင်တွင် Routes ပေါင်းများစွာကို စနစ်တကျ စုစည်းရန် Route Groups ကို မဖြစ်မနေ အသုံးပြုရပါသည်:

```php
Route::middleware(['auth'])               // အဖွဲ့ဝင်များသာ ဝင်ခွင့်ရှိမည်
     ->prefix('admin')                    // URL အရှေ့တွင် /admin/ ပါမည်
     ->name('admin.')                     // Route name အရှေ့တွင် admin. ပါမည်
     ->group(function () {

         // URL: /admin/dashboard | Name: admin.dashboard
         Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');

         // URL: /admin/users | Name: admin.users.index
         Route::get('/users', [UserController::class, 'index'])->name('users.index');

         // URL: /admin/reports | Name: admin.reports
         Route::get('/reports', [ReportController::class, 'index'])->name('reports');
     });
```

---

## ၅။ Controllers အမျိုးအစားများ

Controller ဆိုသည်မှာ HTTP Request ကို လက်ခံပြီး သက်ဆိုင်ရာ Model နှင့် View တို့ကို ချိတ်ဆက်ပေးသော စီမံခန့်ခွဲသူ (Traffic Controller) ဖြစ်သည်။

### (က) Basic Controller
```bash
php artisan make:controller PostController
```

### (ခ) Resource Controller (လုပ်ငန်းခွင်တွင် အသုံးအများဆုံး)
CRUD စနစ်တစ်ခုအတွက် လိုအပ်သော Standard Actions (၇) ခုကို တစ်ပြိုင်နက် ဖန်တီးပေးသည်:
```bash
php artisan make:controller ArticleController --resource
```

`routes/web.php` တွင် လိုင်းတစ်ကြောင်းတည်းဖြင့် CRUD Routes ၇ ခုလုံးကို ချိတ်ဆက်နိုင်သည်:
```php
Route::resource('articles', ArticleController::class);
```

| HTTP Verb | Path | Action | Route Name | ရည်ရွယ်ချက် |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/articles` | `index` | `articles.index` | ဆောင်းပါးအားလုံး စာရင်းပြရန် |
| `GET` | `/articles/create` | `create` | `articles.create` | အသစ်တင်မည့် Form ပြရန် |
| `POST` | `/articles` | `store` | `articles.store` | အသစ်ကို Database သို့ သိမ်းရန် |
| `GET` | `/articles/{article}` | `show` | `articles.show` | ဆောင်းပါးတစ်ခုချင်းစီ အသေးစိတ်ပြရန် |
| `GET` | `/articles/{article}/edit` | `edit` | `articles.edit` | ပြင်ဆင်မည့် Form ပြရန် |
| `PUT/PATCH` | `/articles/{article}` | `update` | `articles.update` | ပြင်ဆင်ချက်ကို Database သို့ update လုပ်ရန် |
| `DELETE` | `/articles/{article}` | `destroy` | `articles.destroy` | ဆောင်းပါးကို ဖျက်ပစ်ရန် |

### (ဂ) Single Action Controller (`__invoke`)
လုပ်ဆောင်ချက်တစ်ခုတည်းသာ သီးသန့်ရှိသောအခါ (ဥပမာ- PDF Export လုပ်ခြင်း):
```bash
php artisan make:controller ExportInvoiceController --invokable
```

```php
class ExportInvoiceController extends Controller
{
    public function __invoke(Invoice $invoice)
    {
        // Invoice PDF ကို generate လုပ်ပြီး download ပေးမည်
    }
}

// routes/web.php
Route::get('/invoices/{invoice}/pdf', ExportInvoiceController::class)->name('invoices.pdf');
```

---

## ၆။ Route Model Binding

**Route Model Binding** သည် URL မှ ပါလာသော ID အလိုက် Database ထဲမှ Model record ကို အလိုအလျောက် Query ဆွဲထုတ်ပေးသော Laravel ၏ အထူးစွမ်းအားမြင့် စနစ်တစ်ခု ဖြစ်သည်။

```php
// ❌ ရိုးရိုး Manual ရေးနည်း (လိုင်းပိုများပြီး 404 ကို ကိုယ်တိုင်စစ်ရသည်)
Route::get('/posts/{id}', function ($id) {
    $post = Post::find($id);
    if (! $post) {
        abort(404);
    }
    return view('posts.show', compact('post'));
});

// ✅ Laravel Implicit Route Model Binding (အလွန် သန့်ရှင်းသည်)
// Parameter name {post} နှင့် Controller argument $post တူညီရပါမည်
Route::get('/posts/{post}', function (Post $post) {
    // Model record မရှိပါက 404 Not Found Page ကို auto ပြပေးသည်
    return view('posts.show', compact('post'));
});
```

#### Slug ဖြင့် Bind လုပ်လိုပါက:
```php
// ID အစား slug ဖြင့် ရှာလိုသောအခါ
Route::get('/posts/{post:slug}', [PostController::class, 'show']);
```

---

## ၇။ လုပ်ငန်းခွင်သုံး CRUD Controller ကုဒ်နမူနာ

```php
namespace App\Http\Controllers;

use App\Models\Article;
use Illuminate\Http\Request;

class ArticleController extends Controller
{
    public function index()
    {
        $articles = Article::latest()->paginate(10);
        return view('articles.index', compact('articles'));
    }

    public function create()
    {
        return view('articles.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'title'   => 'required|max:255',
            'content' => 'required',
        ]);

        Article::create($validated);

        return redirect()->route('articles.index')->with('success', 'ဆောင်းပါး အသစ် တင်ပြီးပါပြီ!');
    }

    public function show(Article $article)
    {
        return view('articles.show', compact('article'));
    }

    public function edit(Article $article)
    {
        return view('articles.edit', compact('article'));
    }

    public function update(Request $request, Article $article)
    {
        $validated = $request->validate([
            'title'   => 'required|max:255',
            'content' => 'required',
        ]);

        $article->update($validated);

        return redirect()->route('articles.index')->with('success', 'ဆောင်းပါး ပြင်ဆင်ပြီးပါပြီ!');
    }

    public function destroy(Article $article)
    {
        $article->delete();
        return redirect()->route('articles.index')->with('success', 'ဆောင်းပါး ဖျက်ပြီးပါပြီ!');
    }
}
```
