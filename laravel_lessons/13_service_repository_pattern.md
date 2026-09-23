# 🏗️ သင်ခန်းစာ (၁၃) - Enterprise Architecture (Service Layer & Repository Pattern)
### (Lesson 13: Clean Architecture, Skinny Controllers, Service Layer & DB Transactions)

---

## 📌 မာတိကာ (Contents)
1. [Fat Controller ပြဿနာနှင့် Clean Architecture လိုအပ်ပုံ](#၁-fat-controller-ပြဿနာနှင့်-clean-architecture)
2. [Database Transactions ၏ အရေးကြီးပုံ (Atomicity)](#၂-database-transactions-၏-အရေးကြီးပုံ)
3. [Service Layer Pattern လက်တွေ့ အကောင်အထည်ဖော်ခြင်း](#၃-service-layer-pattern-လက်တွေ့-အကောင်အထည်ဖော်ခြင်း)
4. [Skinny Controller ဖြင့် ချိတ်ဆက်အသုံးပြုပုံ](#၄-skinny-controller-ဖြင့်-ချိတ်ဆက်အသုံးပြုပုံ)
5. [Repository Pattern ဆိုတာဘာလဲ? မည်သည့်အခါတွင် သုံးသင့်သလဲ?](#၅-repository-pattern-ဆိုတာဘာလဲ)
6. [လုပ်ငန်းခွင် Best Practices အနှစ်ချုပ်](#၆-လုပ်ငန်းခွင်-best-practices-အနှစ်ချုပ်)

---

## ၁။ Fat Controller ပြဿနာနှင့် Clean Architecture

Laravel စတင်လေ့လာသူအများစုသည် Validation, Database Query, Payment Gateway ချိတ်ခြင်း၊ Email ပို့ခြင်း၊ PDF ထုတ်ခြင်း စသည့် အလုပ်အားလုံးကို **Controller တစ်ခုတည်းထဲတွင် ရေးသားလေ့ရှိကြသည်**။ ဤသည်ကို **"Fat Controller"** ဟု ခေါ်ပြီး ကုဒ်များ ရှုပ်ထွေးကာ နောင်တွင် ပြင်ဆင်ရခက်ခဲစေသည်။

### ✅ Enterprise Clean Architecture အလွှာများ:
```
[HTTP Request]
      │
      ▼
[Form Request] ────► Input Validation စစ်ဆေးခြင်း
      │
      ▼
[Controller]   ────► HTTP Flow ကိုသာ စီမံသည် (Skinny Controller)
      │
      ▼
[Service Layer] ───► Business Logic, Payment APIs, Calculations
      │
      ▼
[Eloquent / DB] ───► Database Storage (Transaction Commit/Rollback)
```

---

## ၂။ Database Transactions ၏ အရေးကြီးပုံ

အထူးသဖြင့် E-Commerce နှင့် ငွေကြေးဆိုင်ရာ စနစ်များတွင် Database Operations အဆင့် ၃ ဆင့် လုပ်ရမည်ဆိုပါစို့:
1. Customer အကောင့်မှ ငွေ ၅၀၀၀၀ ဖြတ်မည်။
2. Seller အကောင့်ထဲ ငွေ ၅၀၀၀၀ ထည့်မည်။
3. Order record ကို Database သို့ သိမ်းမည်။

အကယ်၍ အဆင့် (၁) ပြီးပြီး အဆင့် (၂) အရောက်တွင် Server မီးပျက်သွားပါက Customer ထံမှ ငွေဖြတ်ပြီးသော်လည်း Seller ထံ မရောက်သည့် ဆိုးရွားသော အမှား ဖြစ်သွားမည်။

### `DB::transaction()` ၏ ဖြေရှင်းပုံ:
အလုပ်အားလုံး အောင်မြင်မှသာ **Commit** လုပ်ပြီး အမှားတစ်ခုခု ကြုံပါက ယခင် လုပ်ခဲ့သမျှကို အလိုအလျောက် **Rollback** ပြန်ဆုတ်ပေးသည်:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () use ($data) {
    // ဤအကွက်အတွင်း ကုဒ်များအားလုံး အောင်မြင်မှသာ ဒေတာ သိမ်းဆည်းမည်
    // Exception တစ်ခုခု ဖြစ်ပါက အစမှ ပြန်ဆုတ်သွားမည်
});
```

---

## ၃။ Service Layer Pattern လက်တွေ့ အကောင်အထည်ဖော်ခြင်း

E-Commerce Order တင်ခြင်း စနစ်အတွက် Service Class တစ်ခု ဖန်တီးပါမည်: `app/Services/OrderService.php`

```php
namespace App\Services;

use App\Models\Order;
use App\Models\Product;
use App\Models\User;
use Illuminate\Support\Facades\DB;
use Exception;

class OrderService
{
    /**
     * အော်ဒါ အသစ် တည်ဆောက်ခြင်း Business Logic
     */
    public function createOrder(User $user, array $orderData): Order
    {
        return DB::transaction(function () use ($user, $orderData) {
            // ၁။ Order Master Record သိမ်းခြင်း
            $order = Order::create([
                'user_id'      => $user->id,
                'total_amount' => $orderData['total_amount'],
                'status'       => 'pending',
            ]);

            // ၂။ Order Items တစ်ခုချင်းစီ ထည့်သွင်းပြီး Stock လျှော့ချခြင်း
            foreach ($orderData['items'] as $item) {
                $product = Product::findOrFail($item['product_id']);

                // Stock လုံလောက်မှု ရှိမရှိ စစ်ဆေးခြင်း
                if ($product->stock < $item['quantity']) {
                    throw new Exception("ကုန်ပစ္စည်း '{$product->title}' သည် Stock မလုံလောက်ပါ။");
                }

                // Order Items သိမ်းခြင်း
                $order->items()->create([
                    'product_id' => $product->id,
                    'quantity'   => $item['quantity'],
                    'price'      => $product->price,
                ]);

                // Stock အလိုအလျောက် လျှော့ချခြင်း
                $product->decrement('stock', $item['quantity']);
            }

            return $order;
        });
    }
}
```

---

## ၄။ Skinny Controller ဖြင့် ချိတ်ဆက်အသုံးပြုပုံ

Controller ထဲတွင် လိုင်းအနည်းငယ်ဖြင့် သန့်ရှင်းစွာ ရေးသားနိုင်ပါသည်:

```php
namespace App\Http\Controllers;

use App\Http\Requests\StoreOrderRequest;
use App\Services\OrderService;
use Exception;

class OrderController extends Controller
{
    // Dependency Injection ဖြင့် Service ကို Inject လုပ်ခြင်း
    public function __construct(protected OrderService $orderService) {}

    public function store(StoreOrderRequest $request)
    {
        try {
            $order = $this->orderService->createOrder(
                $request->user(),
                $request->validated()
            );

            return response()->json([
                'status'   => 'success',
                'message'  => 'အော်ဒါ အောင်မြင်စွာ တင်ပြီးပါပြီ',
                'order_id' => $order->id,
            ], 201);

        } catch (Exception $e) {
            return response()->json([
                'status'  => 'error',
                'message' => $e->getMessage()
            ], 400);
        }
    }
}
```

---

## ၅။ Repository Pattern ဆိုတာဘာလဲ?

**Repository Pattern** ဆိုသည်မှာ Application Logic နှင့် Database Query များကို ကြားခံ အလွှာတစ်ခုဖြင့် ခွဲထုတ်ထားသော စနစ်ဖြစ်ပါသည်။

```
[Service Layer] ──► [ProductRepositoryInterface] ◄──► [EloquentProductRepository] ──► [Database]
```

### မည်သည့်အခါတွင် သုံးသင့်သလဲ?
* Database Engine ပြောင်းလဲနိုင်ခြေရှိသောအခါ (ဥပမာ- MySQL မှ MongoDB သို့ ပြောင်းနိုင်ခြေ)။
* Unit Testing Mocking ကို အလွန်အမင်း တင်းကြပ်စွာ လုပ်ဆောင်ရသော Enterprise စနစ်ကြီးများတွင်သာ သုံးသင့်သည်။
* ရိုးရိုး Medium Project များတွင် Eloquent Model ကို Service Layer မှ တိုက်ရိုက်ခေါ်သုံးခြင်းသည် ပိုမိုမြန်ဆန်ပြီး Over-engineering မဖြစ်စေပါ။

---

## ၆။ လုပ်ငန်းခွင် Best Practices အနှစ်ချုပ်

1. **Keep Controllers Thin**: Controller ၏ တာဝန်သည် HTTP Request ကို လက်ခံပြီး Response ပြန်ပေးရုံသာ ဖြစ်ရမည်။
2. **Move Logic to Services**: တွက်ချက်မှုများ၊ ပြင်ပ API ခေါ်ယူမှုများအားလုံးကို Service Classes များသို့ ရွှေ့ပါ။
3. **Always Use DB Transactions**: စားပွဲတစ်ခုထက်ပိုသော Database အပြောင်းအလဲများကို Transaction အတွင်း ထည့်သွင်းပါ။
4. **Use Custom Exceptions**: Error များကို စနစ်တကျ ဖမ်းယူနိုင်ရန် Descriptive Exception Messages များကို အသုံးပြုပါ။
