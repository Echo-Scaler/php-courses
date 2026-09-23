# 🌐 သင်ခန်းစာ (၁၀) - RESTful API တည်ဆောက်ခြင်းနှင့် Laravel Sanctum
### (Lesson 10: Building Modern REST APIs, API Resources & Sanctum Token Auth)

---

## 📌 မာတိကာ (Contents)
1. [RESTful API ဆိုတာဘာလဲနှင့် Modern Web/Mobile Architecture](#၁-restful-api-ဆိုတာဘာလဲ)
2. [API Routing နှင့် API Versioning (`routes/api.php`)](#၂-api-routing-နှင့်-api-versioning)
3. [Eloquent API Resources ဖြင့် JSON Output ကို စနစ်တကျ ပြုပြင်ခြင်း](#၃-eloquent-api-resources)
4. [Laravel Sanctum ဖြင့် Token-Based Authentication စနစ် တည်ဆောက်ခြင်း](#၄-laravel-sanctum-ဖြင့်-token-auth)
5. [Login, Register, Logout API Endpoints ရေးသားခြင်း](#၅-login-register-logout-api-endpoints)
6. [Standardized API Response Format နှင့် HTTP Status Codes](#၆-standardized-api-response-format)

---

## ၁။ RESTful API ဆိုတာဘာလဲ?

**RESTful API** ဆိုသည်မှာ Frontend (React, Vue, Next.js) သို့မဟုတ် Mobile App (Flutter, React Native, iOS, Android) များအတွက် HTML စာမျက်နှာများ ပြသမည့်အစား စက်အချင်းချင်း နားလည်နိုင်သော **JSON Format ဒေတာများ** ကို လဲလှယ်ပေးပို့သော Backend စနစ် ဖြစ်သည်။

```
[Mobile App / React SPA]
           │
           │ (1) HTTP Request + Bearer Token (e.g. GET /api/v1/products)
           ▼
┌─────────────────────────────────┐
│     Laravel RESTful API         │
│  • Sanctum Authentication Token │
│  • Eloquent API Resource        │
└──────────┬──────────────────────┘
           │
           │ (2) Pure JSON Data (200 OK)
           ▼
[Mobile App / React UI Rendering]
```

---

## ၂။ API Routing နှင့် API Versioning

Laravel 11 တွင် API routing စတင်အသုံးပြုရန်:
```bash
php artisan install:api
```
ဤ command သည် `routes/api.php` ဖိုင်နှင့် `Laravel Sanctum` ကို အလိုအလျောက် တပ်ဆင်ပေးသည်။

### API Versioning စနစ် (`routes/api.php`):
Mobile App များ Version အသစ်ထွက်တိုင်း Backend မပျက်စီးစေရန် `/api/v1/` စသဖြင့် ခွဲခြားရေးသားခြင်း:

```php
use App\Http\Controllers\Api\V1\ProductController;
use Illuminate\Support\Facades\Route;

Route::prefix('v1')->group(function () {
    // Public Routes (မည်သူမဆို ကြည့်နိုင်သည်)
    Route::get('/products', [ProductController::class, 'index']);
    Route::get('/products/{id}', [ProductController::class, 'show']);

    // Protected Routes (Token ပါမှသာ ခွင့်ပြုမည်)
    Route::middleware('auth:sanctum')->group(function () {
        Route::post('/products', [ProductController::class, 'store']);
        Route::put('/products/{id}', [ProductController::class, 'update']);
        Route::delete('/products/{id}', [ProductController::class, 'destroy']);
    });
});
```

---

## ၃။ Eloquent API Resources

Model မှ Data ကို တိုက်ရိုက် `return $product;` လုပ်မည့်အစား **API Resource** ကို အသုံးပြုရသည်။
* **အားသာချက်**: မလိုအပ်သော Database Column များကို ဖျောက်ထားနိုင်ခြင်း၊ Field အမည်များ ပြောင်းနိုင်ခြင်းနှင့် Date format များကို စနစ်တကျ ပြုပြင်ပေးနိုင်ခြင်း။

```bash
php artisan make:resource ProductResource
```

```php
// app/Http/Resources/ProductResource.php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'              => $this->id,
            'title'           => $this->title,
            'price'           => (float) $this->price,
            'formatted_price' => number_format($this->price) . ' MMK',
            'in_stock'        => $this->stock > 0,
            'category'        => [
                'id'   => $this->category?->id,
                'name' => $this->category?->name,
            ],
            'image_url'       => $this->image ? asset('storage/' . $this->image) : null,
            'created_at'      => $this->created_at->toIso8601String(),
        ];
    }
}
```

---

## ၄။ Laravel Sanctum ဖြင့် Token-Based Authentication

Sanctum သည် SPA နှင့် Mobile App များအတွက် အလွန်ပေါ့ပါးမြန်ဆန်သော Token-based Authentication ကို ပေးစွမ်းသည်။

`app/Models/User.php` တွင် `HasApiTokens` trait ပါဝင်ရမည်:
```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

---

## ၅။ Login, Register, Logout API Endpoints

```php
namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthApiController extends Controller
{
    // ၁။ Register API
    public function register(Request $request)
    {
        $validated = $request->validate([
            'name'     => 'required|string|max:100',
            'email'    => 'required|email|unique:users,email',
            'password' => 'required|min:8',
        ]);

        $user = User::create([
            'name'     => $validated['name'],
            'email'    => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        $token = $user->createToken('mobile-app')->plainTextToken;

        return response()->json([
            'status'  => 'success',
            'message' => 'အကောင့်ဖွင့်ခြင်း အောင်မြင်ပါသည်',
            'data'    => [
                'user'  => $user,
                'token' => $token,
            ]
        ], 201);
    }

    // ၂။ Login API
    public function login(Request $request)
    {
        $request->validate([
            'email'    => 'required|email',
            'password' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (! $user || ! Hash::check($request->password, $user->password)) {
            return response()->json([
                'status'  => 'error',
                'message' => 'Email သို့မဟုတ် စကားဝှက် မှားယွင်းနေပါသည်။'
            ], 401);
        }

        // Bearer Token အသစ် ထုတ်ပေးခြင်း
        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'status'  => 'success',
            'message' => 'Login အောင်မြင်ပါသည်',
            'token'   => $token,
            'user'    => $user,
        ]);
    }

    // ၃။ Logout API
    public function logout(Request $request)
    {
        // လက်ရှိ သုံးနေသော Token ကို ဖျက်သိမ်းခြင်း
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'status'  => 'success',
            'message' => 'Logout အောင်မြင်စွာ ထွက်ပြီးပါပြီ'
        ]);
    }
}
```

---

## ၆။ Standardized API Response Format နှင့် HTTP Status Codes

လုပ်ငန်းခွင်တွင် Frontend Team နှင့် ချိတ်ဆက်အလုပ်လုပ်ရာတွင် Response Format တသမတ်တည်း ရှိရန် အလွန်အရေးကြီးသည်:

### စံ Response ပုံစံ:
```json
{
  "status": "success",
  "message": "Data retrieved successfully",
  "data": { ... }
}
```

| HTTP Status Code | အဓိပ္ပာယ် | မည်သည့်အခါတွင် သုံးသလဲ |
| :--- | :--- | :--- |
| `200 OK` | Success | GET, PUT, PATCH အောင်မြင်သောအခါ |
| `201 Created` | Resource Created | POST ဖြင့် အသစ်တစ်ခု ထည့်သွင်းအောင်မြင်သောအခါ |
| `401 Unauthorized` | Not Logged In | Token မပါခြင်း သို့မဟုတ် Token သက်တမ်းကုန်နေခြင်း |
| `403 Forbidden` | Permission Denied | Login ဝင်ထားသော်လည်း လုပ်ပိုင်ခွင့် မရှိခြင်း |
| `404 Not Found` | Not Found | ရှာဖွေသော Data မရှိသောအခါ |
| `422 Unprocessable`| Validation Failed | Input ဒေတာ မှားယွင်းနေသောအခါ |
| `500 Server Error` | Server Exception | Backend ကုဒ် သို့မဟုတ် Server အမှားဖြစ်သောအခါ |
