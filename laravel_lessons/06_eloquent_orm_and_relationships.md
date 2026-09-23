# 📊 သင်ခန်းစာ (၆) - Eloquent ORM နှင့် Relationships (ဒေတာဘေ့စ် ကျွမ်းကျင်မှု)
### (Lesson 6: Eloquent ORM, CRUD Mastery, Relationships & Solving N+1 Problem)

---

## 📌 မာတိကာ (Contents)
1. [Eloquent ORM ဆိုတာဘာလဲ? (Active Record Pattern)](#၁-eloquent-orm-ဆိုတာဘာလဲ)
2. [Eloquent Model ဖန်တီးခြင်းနှင့် အရေးကြီး Configuration များ](#၂-eloquent-model-ဖန်တီးခြင်းနှင့်-အရေးကြီး-configuration-များ)
3. [Eloquent CRUD Operations (လက်တွေ့ Query ရေးသားနည်းများ)](#၃-eloquent-crud-operations)
4. [Soft Deletes စနစ် (ဒေတာ အပြီးမပျက်ဘဲ သိမ်းဆည်းခြင်း)](#၄-soft-deletes-စနစ်)
5. [Eloquent Relationships (ဇယားများ ဆက်သွယ်ခြင်း)](#၅-eloquent-relationships)
6. [N+1 Query Problem ဆိုတာဘာလဲ? Eager Loading ဖြင့် ဖြေရှင်းပုံ](#၆-n1-query-problem-ဆိုတာဘာလဲ-eager-loading-ဖြင့်-ဖြေရှင်းပုံ)
7. [Query Scopes နှင့် Accessors / Mutators](#၇-query-scopes-နှင့်-accessors--mutators)

---

## ၁။ Eloquent ORM ဆိုတာဘာလဲ?

**Eloquent ORM (Object-Relational Mapper)** သည် Laravel တွင် ပါဝင်သော အစွမ်းထက်ဆုံး Feature တစ်ခု ဖြစ်သည်။ ၎င်းသည် **Active Record Architectural Pattern** ကို အသုံးပြုထားပြီး Database Table တစ်ခုချင်းစီကို PHP Class (Model) တစ်ခုအဖြစ် ပြောင်းလဲပေးသည်။

SQL Raw Query များ (ဥပမာ- `SELECT * FROM users WHERE status = 'active'`) ရေးသားစရာမလိုဘဲ Object Oriented ကုဒ်များဖြင့် ဒေတာများကို လွယ်ကူစွာ စီမံခန့်ခွဲနိုင်ပါသည်။

---

## ၂။ Eloquent Model ဖန်တီးခြင်းနှင့် အရေးကြီး Configuration များ

```bash
# Model တစ်ခု ဆောက်ခြင်း
php artisan make:model Product
```

```php
// app/Models/Product.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Product extends Model
{
    use SoftDeletes; // deleted_at column ကို auto စီမံပေးသည်

    // မူလ Table အမည် (သတ်မှတ်မထားပါက products ဟု auto ယူသည်)
    protected $table = 'products';

    // Mass Assignment Protection: User တိုက်ရိုက် ထည့်ခွင့်ပြုမည့် Columns များ
    protected $fillable = [
        'category_id',
        'title',
        'slug',
        'price',
        'stock',
        'is_active',
    ];

    // Array / JSON ပြောင်းသည့်အခါ ဖျောက်ထားမည့် Columns များ
    protected $hidden = [
        'cost_price',
    ];

    // Data Types များကို အလိုအလျောက် သင့်တော်သော PHP Type သို့ ပြောင်းပေးခြင်း
    protected $casts = [
        'price'      => 'decimal:2',
        'stock'      => 'integer',
        'is_active'  => 'boolean',
        'created_at' => 'datetime',
    ];
}
```

---

## ၃။ Eloquent CRUD Operations

### (က) Create (ဒေတာ အသစ် ထည့်သွင်းခြင်း)
```php
// နည်းလမ်း ၁: create() method (Mass Assignment)
$product = Product::create([
    'category_id' => 1,
    'title'       => 'iPhone 15 Pro',
    'price'       => 3800000,
    'stock'       => 15,
]);

// နည်းလမ်း ၂: Object save() method
$product = new Product();
$product->title = 'Samsung Galaxy S24';
$product->price = 3500000;
$product->save();
```

### (ခ) Read (ဒေတာများ ရှာဖွေရယူခြင်း)
```php
// အားလုံး ဆွဲထုတ်ခြင်း
$allProducts = Product::all();

// ID ဖြင့် ရှာခြင်း (မရှိပါက 404 Page ကို အလိုအလျောက် ပစ်ပေးသည်)
$product = Product::findOrFail($id);

// Conditions များနှင့် စစ်ထုတ်ခြင်း
$featuredProducts = Product::where('price', '>', 500000)
                           ->where('is_active', true)
                           ->orderBy('created_at', 'desc')
                           ->take(5)
                           ->get();

// Pagination ဖြင့် စာမျက်နှာခွဲခြင်း (လုပ်ငန်းခွင်တွင် အမြဲသုံးသည်)
$paginatedProducts = Product::latest()->paginate(15);
```

### (ဂ) Update (ဒေတာ ပြင်ဆင်ခြင်း)
```php
$product = Product::findOrFail($id);
$product->update([
    'price' => 3600000,
    'stock' => 10,
]);
```

### (ဃ) Delete (ဒေတာ ဖျက်ခြင်း)
```php
$product = Product::findOrFail($id);
$product->delete(); // Soft delete ဖြစ်သွားသည်

// ID ပေးပြီး တိုက်ရိုက်ဖျက်ခြင်း
Product::destroy([1, 2, 3]);
```

---

## ၄။ Soft Deletes စနစ်

Model တွင် `SoftDeletes` ထည့်ထားပါက `delete()` ခေါ်လိုက်သည့်အခါ Database ထဲမှ အပြီးတိုင် မပျက်သွားဘဲ `deleted_at` column တွင် လက်ရှိအချိန် တံဆိပ်ရိုက်သွားမည် ဖြစ်သည်။

```php
// ပုံမှန် Query လုပ်ပါက Soft-deleted record များ ပါမလာတော့ပါ
$products = Product::all();

// အမှိုက်ပုံးထဲ ရောက်နေသော ဒေတာများပါ ပြန်ဖတ်လိုပါက:
$allWithTrash = Product::withTrashed()->get();

// ဖျက်ထားသော ဒေတာ သီးသန့် ရှာလိုပါက:
$trashedOnly = Product::onlyTrashed()->get();

// အမှားပြင်ဆင်ပြီး ပြန်လည် အသက်သွင်းခြင်း (Restore):
$product->restore();

// Database ထဲမှ အပြီးတိုင် ဖျက်ပစ်ခြင်း:
$product->forceDelete();
```

---

## ၅။ Eloquent Relationships

### (က) One-to-Many (1:N)
Category တစ်ခုတွင် Products များစွာ ရှိသည်။ Product တစ်ခုသည် Category တစ်ခုအောက်တွင်သာ ရှိသည်။

```php
// app/Models/Category.php
public function products()
{
    return $this->hasMany(Product::class);
}

// app/Models/Product.php
public function category()
{
    return $this->belongsTo(Category::class);
}

// သုံးစွဲပုံ:
$category = Category::find(1);
foreach ($category->products as $product) {
    echo $product->title;
}
```

### (ခ) Many-to-Many (N:N)
Order တစ်ခုတွင် Products များစွာ ပါဝင်နိုင်ပြီး Product တစ်ခုသည် Orders များစွာတွင် ပါဝင်နိုင်သည်။ (Pivot Table: `order_items`)

```php
// app/Models/Order.php
public function products()
{
    return $this->belongsToMany(Product::class, 'order_items')
                ->withPivot('quantity', 'unit_price')
                ->withTimestamps();
}

// Attach လုပ်ခြင်း (Order ထဲ Product ထည့်ခြင်း):
$order->products()->attach($productId, ['quantity' => 2, 'unit_price' => 1500]);

// Sync လုပ်ခြင်း (Array အတိုင်းသာ ထားပြီး ကျန်တာ auto ဖြုတ်ခြင်း):
$order->products()->sync([1 => ['quantity' => 1], 2 => ['quantity' => 3]]);
```

---

## ၆။ N+1 Query Problem ဆိုတာဘာလဲ? Eager Loading ဖြင့် ဖြေရှင်းပုံ

**N+1 Problem** သည် Laravel စတင်လေ့လာသူများ Database ကို အလွန်လေးလံစေသည့် အဓိက အမှားတစ်ခု ဖြစ်သည်။

```php
// ❌ ဆိုးရွားသော ရေးသားမှု (Lazy Loading - N+1 Problem ဖြစ်သည်)
$products = Product::all(); // Query (1) ကြိမ် run သည်
foreach ($products as $product) {
    echo $product->category->name; // Loop အကြိမ်တိုင်း DB သို့ Query 1 ခုစီ ထပ်ခေါ်သည် (N ကြိမ်)
}
// အကယ်၍ Product အခု ၁၀၀ ရှိပါက Database Query ပေါင်း ၁၀၁ ကြိမ် ထိခိုက်သွားပါမည်!
```

```php
// ✅ အကောင်းဆုံး ရေးသားမှု (Eager Loading - with() အသုံးပြုခြင်း)
$products = Product::with('category')->get(); // Query (၂) ကြိမ်သာ run သည်!
foreach ($products as $product) {
    echo $product->category->name; // Memory ထဲမှသာ ချက်ချင်းယူသုံးသည်
}
```

---

## ၇။ Query Scopes နှင့် Accessors / Mutators

### (က) Local Query Scopes (ထပ်ခါတလဲလဲ သုံးရသော Query များကို စုစည်းခြင်း)
```php
// app/Models/Product.php
public function scopePopular($query)
{
    return $query->where('stock', '>', 0)->where('is_active', true);
}

// Controller တွင် သန့်ရှင်းစွာ သုံးစွဲပုံ:
$popularProducts = Product::popular()->latest()->get();
```

### (ခ) Accessors (Data ကို ဆွဲထုတ်ချိန်တွင် ပြုပြင်ပြသခြင်း)
```php
use Illuminate\Database\Eloquent\Casts\Attribute;

// $product->formatted_price ဟု ခေါ်ယူနိုင်သည်
protected function formattedPrice(): Attribute
{
    return Attribute::make(
        get: fn () => number_format($this->price) . ' MMK'
    );
}
```
