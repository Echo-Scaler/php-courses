# 🛠️ သင်ခန်းစာ (၁၂) - လုပ်ငန်းခွင်သုံး Built-in Helpers, Collections နှင့် Facades
### (Lesson 12: Master Laravel Collections, String/Array Helpers, Carbon & Facades)

---

## 📌 မာတိကာ (Contents)
1. [Facades နှင့် Helper Functions သဘောတရား](#၁-facades-နှင့်-helper-functions-သဘောတရား)
2. [Laravel Collections စွမ်းအားနှင့် မဖြစ်မနေသိရမည့် Methods များ](#၂-laravel-collections-စွမ်းအား)
3. [String Helpers (`Str` Facade) အသုံးချပုံများ](#၃-string-helpers-str-facade)
4. [Array Helpers (`Arr` Facade) အသုံးချပုံများ](#၄-array-helpers-arr-facade)
5. [Carbon ဖြင့် Date & Time တွက်ချက်စီမံခြင်း](#၅-carbon-ဖြင့်-date--time-တွက်ချက်စီမံခြင်း)
6. [နေ့စဉ် မသုံးမဖြစ် General Helper Functions စာရင်း](#၆-နေ့စဉ်-မသုံးမဖြစ်-general-helper-functions)

---

## ၁။ Facades နှင့် Helper Functions သဘောတရား

* **Facades**: Laravel ၏ Service Container ထဲရှိ Class များကို Static Interface ပုံစံဖြင့် လွယ်ကူစွာ ခေါ်ယူအသုံးပြုနိုင်စေရန် ဖန်တီးထားသော စနစ်ဖြစ်သည် (ဥပမာ- `Cache::get()`, `Route::get()`, `DB::table()`, `Log::info()`)။
* **Global Helpers**: မည်သည့်နေရာ (Blade, Controller, Model) တွင်မဆို `use` မလိုဘဲ တိုက်ရိုက် ခေါ်သုံးနိုင်သော PHP functions များဖြစ်သည် (ဥပမာ- `now()`, `dd()`, `redirect()`, `auth()`)။

---

## ၂။ Laravel Collections စွမ်းအား

PHP ၏ ရိုးရိုး Array များဖြင့် အလုပ်လုပ်ရသည်ထက် Laravel ၏ **Collection Wrapper** သည် Data များကို စစ်ထုတ်တွက်ချက်ရာတွင် အဆပေါင်းများစွာ ပိုမိုစွမ်းအားထက်မြက်သည်။

```php
$products = collect([
    ['id' => 1, 'name' => 'MacBook Air', 'price' => 1200, 'category' => 'Laptop', 'active' => true],
    ['id' => 2, 'name' => 'Dell XPS',     'price' => 1500, 'category' => 'Laptop', 'active' => false],
    ['id' => 3, 'name' => 'Magic Mouse',  'price' => 80,   'category' => 'Accessory', 'active' => true],
    ['id' => 4, 'name' => 'Keychron K2',  'price' => 100,  'category' => 'Accessory', 'active' => true],
]);
```

### အသုံးများသော Collection Methods:
```php
// ၁။ filter(): သတ်မှတ်ချက်နှင့် ကိုက်ညီသော ဒေတာများသာ ရွေးထုတ်ခြင်း
$activeItems = $products->filter(fn($item) => $item['active']);

// ၂။ pluck(): သီးသန့် Column တစ်ခုတည်းကိုသာ Array အဖြစ် ယူခြင်း
$names = $products->pluck('name');
// ရလဒ်: ['MacBook Air', 'Dell XPS', 'Magic Mouse', 'Keychron K2']

// ၃။ sum() နှင့် avg(): ပေါင်းလဒ်နှင့် ပျမ်းမျှ တွက်ချက်ခြင်း
$totalRevenue = $products->sum('price'); // 2880
$averagePrice = $products->avg('price'); // 720

// ၄။ groupBy(): အုပ်စုခွဲခြင်း
$grouped = $products->groupBy('category');
// ရလဒ်: Laptop အုပ်စုတစ်ခု၊ Accessory အုပ်စုတစ်ခု ခွဲထုတ်ပေးသည်

// ၅။ map(): ဒေတာတစ်ခုချင်းစီကို ပြုပြင်ပြောင်းလဲခြင်း
$withTax = $products->map(function ($item) {
    $item['price_with_tax'] = $item['price'] * 1.05;
    return $item;
});

// ၆။ firstWhere(): ပထမဆုံး ကိုက်ညီသည့် record တစ်ခုတည်းကို ရှာခြင်း
$found = $products->firstWhere('name', 'Magic Mouse');

// ၇။ chunk(): ဒေတာပမာဏများပြားပါက အစိတ်စိတ်ခွဲ၍ စီမံခြင်း
$chunks = $products->chunk(2);
```

---

## ၃။ String Helpers (`Str` Facade)

```php
use Illuminate\Support\Str;

// ၁။ URL Slug ဖန်တီးခြင်း (စာလုံးအသေးပြောင်းပြီး space များကို dash ဖြင့် အစားထိုးခြင်း)
$slug = Str::slug('Mastering Laravel Framework In 2026');
// ရလဒ်: 'mastering-laravel-framework-in-2026'

// ၂။ စာလုံးရေ အရှည်ဖြတ်တောက်ခြင်း (Excerpt / Summary အတွက်)
$excerpt = Str::limit('ဤသင်တန်းသည် အလွန်အသုံးဝင်သော Laravel လမ်းညွှန်ဖြစ်ပြီး...', 20, '...');

// ၃။ စာသား ပါဝင်မှု ရှိမရှိ စစ်ဆေးခြင်း
if (Str::contains('admin@gmail.com', 'admin')) {
    // true
}

// ၄။ Unique UUID နှင့် Random Strings ထုတ်ခြင်း
$uuid = Str::uuid();      // e.g. "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d"
$code = Str::random(8);   // e.g. "aX9L2pQz"

// ၅။ စကားလုံး အနည်း/အများ ပြောင်းခြင်း (Pluralization)
$plural = Str::plural('category'); // 'categories'
$singular = Str::singular('users'); // 'user'

// ၆။ လျှို့ဝှက်အချက်အလက်များကို ကြယ်ပွင့်ဖုံးအုပ်ခြင်း (Masking)
$maskedEmail = Str::mask('john.doe@example.com', '*', 3, 5);
// ရလဒ်: 'joh*****@example.com'
```

---

## ၄။ Array Helpers (`Arr` Facade)

```php
use Illuminate\Support\Arr;

$data = [
    'user' => [
        'name' => 'Mg Mg',
        'address' => [
            'city' => 'Yangon',
            'township' => 'Hledan'
        ]
    ],
    'role' => 'editor'
];

// ၁။ Dot Notation ဖြင့် Nested Array ထဲမှ ဒေတာ အလွယ်တကူ ဆွဲထုတ်ခြင်း
$city = Arr::get($data, 'user.address.city', 'Default City'); // 'Yangon'

// ၂။ လိုအပ်သော Keys သာ ရွေးထုတ်ခြင်း
$only = Arr::only($data, ['role']);

// ၃။ မလိုလားအပ်သော Keys များကို ဖယ်ထုတ်ခြင်း
$except = Arr::except($data, ['role']);

// ၄။ Key ပါမပါ စစ်ဆေးခြင်း
if (Arr::has($data, 'user.address.township')) {
    // true
}
```

---

## ၅။ Carbon ဖြင့် Date & Time တွက်ချက်စီမံခြင်း

Laravel တွင် **Carbon** Library ပါဝင်သောကြောင့် အချိန်နှင့် ရက်စွဲများကို အလွန်စွမ်းအားပြည့် ကိုင်တွယ်နိုင်သည်:

```php
use Carbon\Carbon;

// လက်ရှိ အချိန်နှင့် နေ့စွဲ ရယူခြင်း
$now = now(); // Helper
$today = today(); // Helper (Time is 00:00:00)

// လူနားလည်လွယ်သော အချိန်ဖော်ပြချက် (e.g. Facebook style "2 hours ago")
$postTime = Carbon::parse('2026-09-23 10:00:00');
echo $postTime->diffForHumans(); // "3 hours ago" သို့မဟုတ် "5 days ago"

// နေ့ရက်များ ပေါင်းခြင်း / နှုတ်ခြင်း
$expireDate = now()->addDays(30);       // ရက်ပေါင်း ၃၀ အကြာ
$pastMonth  = now()->subMonths(3);      // လွန်ခဲ့သော ၃ လ

// အချိန်အတိတ်/အနာဂတ် စစ်ဆေးခြင်း
if ($expireDate->isFuture()) {
    echo "သက်တမ်း မကုန်သေးပါ";
}
if ($pastMonth->isPast()) {
    echo "အချိန် ကုန်လွန်သွားပါပြီ";
}

// Format ပြောင်းခြင်း
echo now()->format('Y-m-d H:i A'); // "2026-09-23 13:45 PM"
```

---

## ၆။ နေ့စဉ် မသုံးမဖြစ် General Helper Functions

| Helper | လုပ်ဆောင်ချက်နှင့် ကုဒ်နမူနာ |
| :--- | :--- |
| `dd($val)` | **Dump and Die**: ကုဒ်ကို ရပ်တန့်ပြီး Variable တန်ဖိုးကို Debug ကြည့်ခြင်း |
| `dump($val)` | ကုဒ်မရပ်ဘဲ တန်ဖိုးကို Screen ပေါ် ဖော်ပြခြင်း |
| `logger('Log text')` | `storage/logs/laravel.log` ထဲ အချက်အလက် သိမ်းဆည်းခြင်း |
| `auth()->user()` | လက်ရှိ Login ဝင်ထားသော User Object ကို ယူခြင်း |
| `redirect()->back()` | ယခင် မူလ စာမျက်နှာဟောင်းသို့ ပြန်ပို့ခြင်း |
| `redirect()->route('name')` | သတ်မှတ်ထားသော Named Route ဆီသို့ ပို့ဆောင်ခြင်း |
| `abort(404, 'Msg')` | HTTP Error Page အား ချက်ချင်း ထုတ်ပြခြင်း |
| `config('app.timezone')` | Configuration တန်ဖိုး ဖတ်ယူခြင်း |
| `asset('css/app.css')` | Public directory ထဲရှိ File URL အပြည့်အစုံ ရယူခြင်း |
