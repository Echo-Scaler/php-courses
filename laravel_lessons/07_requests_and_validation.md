# 🛡️ သင်ခန်းစာ (၇) - Request Data ဖမ်းယူခြင်းနှင့် Form Request Validation
### (Lesson 7: Handling HTTP Requests, Form Request Classes & Validation Rules)

---

## 📌 မာတိကာ (Contents)
1. [Request Object ဆိုတာဘာလဲနှင့် Data ဖမ်းယူပုံများ](#၁-request-object-ဆိုတာဘာလဲနှင့်-data-ဖမ်းယူပုံများ)
2. [Controller ထဲတွင် Inline Validation စစ်ဆေးခြင်း](#၂-controller-ထဲတွင်-inline-validation-စစ်ဆေးခြင်း)
3. [Dedicated Form Request Class အသုံးပြုခြင်း (Enterprise Best Practice)](#၃-dedicated-form-request-class-အသုံးပြုခြင်း)
4. [လုပ်ငန်းခွင်တွင် အသုံးအများဆုံး Validation Rules များ](#၄-လုပ်ငန်းခွင်တွင်-အသုံးအများဆုံး-validation-rules-များ)
5. [မြန်မာလို Custom Error Messages ရေးသားသတ်မှတ်ခြင်း](#၅-မြန်မာလို-custom-error-messages-ရေးသားသတ်မှတ်ခြင်း)
6. [Blade View နှင့် API JSON များတွင် Validation Errors ပြသပုံ](#၆-blade-view-နှင့်-api-json-များတွင်-validation-errors-ပြသပုံ)

---

## ၁။ Request Object ဆိုတာဘာလဲနှင့် Data ဖမ်းယူပုံများ

အသုံးပြုသူ Browser မှ Form submit လုပ်လိုက်သော ဒေတာများ၊ Query String (URL parameters) များနှင့် Headers များကို Laravel ၏ `Illuminate\Http\Request` Class ဖြင့် ဖမ်းယူရရှိနိုင်ပါသည်။

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function store(Request $request)
    {
        // ၁။ Input တန်ဖိုး တစ်ခုချင်း ဖမ်းယူခြင်း
        $name = $request->input('name');
        $email = $request->input('email', 'default@example.com'); // မပါလာပါက default တန်ဖိုး ယူမည်

        // ၂။ အားလုံးကို Array ဖြင့် ရယူခြင်း
        $allData = $request->all();

        // ၃။ သီးသန့် လိုအပ်သော Field သာ ရွေးထုတ်ယူခြင်း (အလွန် လုံခြုံသည်)
        $userData = $request->only(['name', 'email', 'phone']);

        // ၄။ မလိုချင်သော Field များကို ချန်လှပ်ယူခြင်း
        $cleanData = $request->except(['_token', 'password_confirmation']);

        // ၅။ Field ပါမပါ စစ်ဆေးခြင်း
        if ($request->has('newsletter')) {
            // Field ပါဝင်သည် (တန်ဖိုး null ဖြစ်နေရင်လည်း true)
        }
        if ($request->filled('username')) {
            // Field ပါဝင်ပြီး တန်ဖိုး ကွက်လပ်မဟုတ်မှသာ true
        }

        // ၆။ URL Query String ဖမ်းယူခြင်း (e.g. ?page=2&sort=asc)
        $page = $request->query('page', 1);

        // ၇။ IP Address နှင့် User Agent ရယူခြင်း
        $ip = $request->ip();
        $userAgent = $request->userAgent();
    }
}
```

---

## ၂။ Controller ထဲတွင် Inline Validation စစ်ဆေးခြင်း

ဒေတာ မသိမ်းဆည်းမီ မဖြစ်မနေ အချက်အလက် မှန်/မမှန် စစ်ဆေးရပါသည်:

```php
public function register(Request $request)
{
    $validated = $request->validate([
        'name'     => 'required|string|max:50',
        'email'    => 'required|email|unique:users,email',
        'password' => 'required|min:8|confirmed', // password_confirmation နှင့် တိုက်စစ်သည်
        'age'      => 'nullable|integer|min:18',
    ]);

    // Validation ကျရှုံးပါက အောက်ကုဒ်များ ဆက်မ run တော့ဘဲ မူလ စာမျက်နှာသို့ Error များနှင့်အတူ အလိုအလျောက် ပြန်ပို့ပေးသည်
    User::create($validated);

    return redirect()->route('home')->with('success', 'အကောင့်ဖွင့်ခြင်း အောင်မြင်ပါသည်');
}
```

---

## ၃။ Dedicated Form Request Class အသုံးပြုခြင်း

Controller ထဲတွင် Validation Rules များ များပြားလာပါက Controller ဖိုင်သည် ကြီးမားရှုပ်ထွေးလာသည်။ ထို့ကြောင့် **Form Request** သီးသန့် Class ခွဲထုတ်ခြင်းသည် လုပ်ငန်းခွင် စံသတ်မှတ်ချက် (Best Practice) ဖြစ်သည်။

```bash
php artisan make:request StoreProductRequest
```

```php
// app/Http/Requests/StoreProductRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreProductRequest extends FormRequest
{
    // ၁။ ဤ Form ကို ပို့ခွင့်ရှိ/မရှိ စစ်ဆေးခြင်း (Authorization)
    public function authorize(): bool
    {
        // ဥပမာ- Admin ဖြစ်မှသာ Product သွင်းခွင့်ပြုမည်
        return true; 
    }

    // ၂။ Validation စည်းကမ်းချက်များ
    public function rules(): array
    {
        return [
            'category_id' => ['required', 'exists:categories,id'],
            'title'       => ['required', 'string', 'max:255'],
            'price'       => ['required', 'numeric', 'min:0'],
            'image'       => ['nullable', 'image', 'mimes:jpeg,png,webp', 'max:2048'], // 2MB Max
        ];
    }
}
```

#### Controller တွင် သုံးစွဲပုံ:
```php
public function store(StoreProductRequest $request)
{
    // Request ကို Inject လုပ်ထားရုံဖြင့် Validation အလိုအလျောက် စစ်ပြီးသား ဖြစ်သည်
    $product = Product::create($request->validated());

    return redirect()->route('products.index');
}
```

---

## ၄။ လုပ်ငန်းခွင်တွင် အသုံးအများဆုံး Validation Rules များ

| Rule | အဓိပ္ပာယ်နှင့် အသုံးပြုပုံ |
| :--- | :--- |
| `required` | ကွက်လပ် လုံးဝ မဖြစ်ရပါ (မဖြစ်မနေ ထည့်ရမည်) |
| `nullable` | တန်ဖိုး မထည့်ဘဲ ကွက်လပ်ထားခွင့်ပြုသည် |
| `email` | မှန်ကန်သော Email format ဖြစ်ရမည် |
| `unique:table,column` | Database table ထဲတွင် တန်ဖိုး ထပ်နေ၍ မရပါ (e.g. `unique:users,email`) |
| `exists:table,column` | Database table ထဲတွင် အဆိုပါ ID/တန်ဖိုး အမှန်တကယ် ရှိနေရမည် |
| `confirmed` | Form တွင် `field_confirmation` (ဥပမာ password_confirmation) နှင့် တန်ဖိုးတူရမည် |
| `min:value` / `max:value` | String ဆိုလျှင် စာလုံးရေ အနည်း/အများ၊ Number ဆိုလျှင် တန်ဖိုး ပမာဏ |
| `numeric` / `integer` | ဂဏန်း သီးသန့် ဖြစ်ရမည် |
| `image` | ပုံဖိုင် (jpg, jpeg, png, bmp, gif, svg, webp) သာ ဖြစ်ရမည် |
| `mimes:pdf,docx` | သတ်မှတ်ထားသော ဖိုင် extension သာ ဖြစ်ရမည် |

---

## ၅။ မြန်မာလို Custom Error Messages ရေးသားသတ်မှတ်ခြင်း

Form Request ဖိုင်ထဲတွင် `messages()` method ကို override လုပ်၍ မြန်မာဘာသာဖြင့် သတိပေးစာများ ရေးသားနိုင်ပါသည်:

```php
public function messages(): array
{
    return [
        'category_id.required' => 'ကုန်ပစ္စည်း အမျိုးအစား ရွေးချယ်ပေးရန် လိုအပ်ပါသည်။',
        'category_id.exists'   => 'ရွေးချယ်ထားသော အမျိုးအစား မရှိပါ။',
        'title.required'       => 'ကုန်ပစ္စည်း အမည် ထည့်သွင်းပေးပါ။',
        'title.max'            => 'အမည်သည် စာလုံးရေ ၂၅၅ ထက် မကျော်ရပါ။',
        'price.required'       => 'ဈေးနှုန်း ထည့်သွင်းပေးပါ။',
        'price.numeric'        => 'ဈေးနှုန်းသည် ဂဏန်း ဖြစ်ရပါမည်။',
        'image.image'          => 'ပုံဖိုင်သာ တင်ပေးရပါမည်။',
        'image.max'            => 'ပုံဖိုင် အရွယ်အစားသည် 2MB ထက် မကြီးရပါ။',
    ];
}
```

---

## ၆။ Blade View နှင့် API JSON များတွင် Validation Errors ပြသပုံ

### (က) Blade Template ထဲတွင် ပြသပုံ:
```html
<div class="form-group">
    <label for="title">ပစ္စည်းအမည်:</label>
    {{-- old('title') ဖြင့် ယခင် ရိုက်ထည့်ခဲ့သော စာသားကို ပြန်ထိန်းထားပေးသည် --}}
    <input type="text" name="title" id="title" value="{{ old('title') }}" 
           class="@error('title') is-invalid @enderror">

    {{-- Error Message သီးသန့် ပြသခြင်း --}}
    @error('title')
        <div class="text-danger">{{ $message }}</div>
    @enderror
</div>
```

### (ခ) RESTful API တွင် အလိုအလျောက် တုံ့ပြန်ပုံ (HTTP 422 Unprocessable Entity):
API Route ဖြစ်ပါက Laravel က အောက်ပါ JSON ပုံစံဖြင့် Status 422 ဖြင့် အလိုအလျောက် ပြန်ပေးသည်:
```json
{
  "message": "ကုန်ပစ္စည်း အမည် ထည့်သွင်းပေးပါ။ (and 1 other error)",
  "errors": {
    "title": [
      "ကုန်ပစ္စည်း အမည် ထည့်သွင်းပေးပါ။"
    ],
    "price": [
      "ဈေးနှုန်း ထည့်သွင်းပေးပါ။"
    ]
  }
}
```
