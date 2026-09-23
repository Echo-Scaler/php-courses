# 🗄️ သင်ခန်းစာ (၅) - Database Migrations, Seeders နှင့် Factories
### (Lesson 5: Database Schema Design, Migrations, Seeders & Faker Factories)

---

## 📌 မာတိကာ (Contents)
1. [Database Migration ဆိုတာဘာလဲ? (Version Control for Database)](#၁-database-migration-ဆိုတာဘာလဲ)
2. [Migration ဖိုင်တစ်ခု၏ ဖွဲ့စည်းပုံ (`up` နှင့် `down`)](#၂-migration-ဖိုင်တစ်ခု၏-ဖွဲ့စည်းပုံ)
3. [Column Types များနှင့် Modifiers များ](#၃-column-types-များနှင့်-modifiers-များ)
4. [Foreign Key Relationships နှင့် Cascading Rules](#၄-foreign-key-relationships-နှင့်-cascading-rules)
5. [ရှိပြီးသား Table တွင် Column အသစ် ထပ်တိုးခြင်း (Safe Migration)](#၅-ရှိပြီးသား-table-တွင်-column-အသစ်-ထပ်တိုးခြင်း)
6. [Seeders နှင့် Model Factories (Faker ဖြင့် Dummy Data ထည့်သွင်းခြင်း)](#၆-seeders-နှင့်-model-factories)
7. [မကြာခဏသုံးရသော Artisan Database Commands](#၇-မကြာခဏသုံးရသော-artisan-database-commands)

---

## ၁။ Database Migration ဆိုတာဘာလဲ?

**Database Migration** ဆိုသည်မှာ Database GUI (phpMyAdmin/Navicat) ထဲတွင် Table များကို လက်ဖြင့် ဆောက်မည့်အစား **PHP ကုဒ်များဖြင့် Version Control ပြုလုပ်ကာ Database Schema ကို တည်ဆောက်ခြင်း** ဖြစ်ပါသည်။

### Migration အသုံးပြုခြင်း၏ အကျိုးကျေးဇူးများ:
1. **Team Collaboration**: အဖွဲ့သားတစ်ဦးက Table သို့မဟုတ် Column အသစ် ထပ်တိုးလိုက်ပါက `php artisan migrate` run လိုက်ရုံဖြင့် အခြား အဖွဲ့သားအားလုံးထံတွင် Database structure အတူတူ ချက်ချင်း ဖြစ်သွားသည်။
2. **Rollback ပြုလုပ်နိုင်ခြင်း**: အမှားတစ်ခုခုပါသွားပါက နောက်ဆုံး ပြုလုပ်ခဲ့သော အပြောင်းအလဲကို အလွယ်တကူ ပြန်ဖျက်နိုင်သည်။
3. **Database Agnostic**: MySQL, PostgreSQL, SQLite, SQL Server စသည့် မည်သည့် Database Engine ကိုမဆို ကုဒ်မပြောင်းဘဲ ချိတ်ဆက်အသုံးပြုနိုင်သည်။

---

## ၂။ Migration ဖိုင်တစ်ခု၏ ဖွဲ့စည်းပုံ

```bash
# Migration အသစ် ဖန်တီးခြင်း
php artisan make:migration create_categories_table
```

Migration ဖိုင်တွင် အဓိက Method (၂) ခု ပါဝင်သည်:
1. `up()`: Table အသစ် ဆောက်ခြင်း သို့မဟုတ် Column များ ထပ်ထည့်ခြင်း။
2. `down()`: `up()` တွင် လုပ်ခဲ့သမျှကို ပြောင်းပြန် ပြန်ဖျက်ပစ်ခြင်း (Rollback)။

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();                     // Auto-increment Primary Key (BIGINT)
            $table->string('name', 100);      // VARCHAR(100)
            $table->string('slug')->unique(); // Unique Index ပါဝင်သည်
            $table->text('description')->nullable(); // NULL တန်ဖိုး ခွင့်ပြုသည်
            $table->boolean('is_active')->default(true); // Default 1
            $table->timestamps();             // created_at & updated_at columns
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('categories');
    }
};
```

---

## ၃။ Column Types များနှင့် Modifiers များ

### (က) အသုံးများသော Column Types:
| Method | MySQL Data Type | ဖော်ပြချက် |
| :--- | :--- | :--- |
| `$table->string('title', 255)` | `VARCHAR(255)` | စာတိုများ (အမည်၊ ခေါင်းစဉ်) |
| `$table->text('body')` | `TEXT` | စာရှည်များ (ဆောင်းပါး၊ မှတ်ချက်) |
| `$table->integer('votes')` | `INT` | ကိန်းပြည့် ဂဏန်းများ |
| `$table->decimal('price', 12, 2)` | `DECIMAL(12,2)` | ငွေကြေး ပမာဏ (အပြည့် ၁၂ လုံး၊ ဒဿမ ၂ လုံး) |
| `$table->boolean('is_published')`| `TINYINT(1)` | True / False တန်ဖိုး |
| `$table->json('settings')` | `JSON` | JSON ဒေတာများ သိမ်းဆည်းရန် |
| `$table->date('birth_date')` | `DATE` | နေ့စွဲ (YYYY-MM-DD) |
| `$table->softDeletes()` | `TIMESTAMP` | `deleted_at` column ထည့်သွင်းခြင်း |

### (ခ) အသုံးများသော Column Modifiers:
* `->nullable()` : ကွက်လပ်ထားခွင့်ပြုခြင်း။
* `->default($value)` : မူလသတ်မှတ်တန်ဖိုး။
* `->unique()` : တန်ဖိုး ထပ်မနေစေရန် ကာကွယ်ခြင်း။
* `->index()` : Query ရှာဖွေမှု ပိုမြန်စေရန် Index သတ်မှတ်ခြင်း။

---

## ၄။ Foreign Key Relationships နှင့် Cascading Rules

Table (၂) ခု ချိတ်ဆက်ရာတွင် Foreign Key သတ်မှတ်ရန် Laravel တွင် အလွန်ရှင်းလင်းသော syntax ရှိပါသည်:

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    
    // categories table ၏ id column နှင့် ချိတ်ဆက်ခြင်း
    // အကယ်၍ category ဖျက်လိုက်ပါက ၎င်းနှင့်ဆိုင်သော products အားလုံး အလိုအလျောက် ပျက်သွားမည် (cascadeOnDelete)
    $table->foreignId('category_id')
          ->constrained('categories')
          ->cascadeOnDelete();

    $table->string('title');
    $table->decimal('price', 10, 2);
    $table->timestamps();
});
```

---

## ၅။ ရှိပြီးသား Table တွင် Column အသစ် ထပ်တိုးခြင်း

Production ရှိပြီးသား ဒေတာများ မပျက်စီးစေရန် Table ကို အသစ်ပြန်မဖျက်ဘဲ သီးသန့် migration တစ်ခု ထပ်ဆောက်ရပါသည်:

```bash
php artisan make:migration add_phone_to_users_table --table=users
```

```php
return new class extends Migration {
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            // email နောက်တွင် phone column ကို ထည့်သွင်းမည်
            $table->string('phone', 20)->nullable()->after('email');
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            // rollback လုပ်ပါက phone column ကို ဖျက်မည်
            $table->dropColumn('phone');
        });
    }
};
```

---

## ၆။ Seeders နှင့် Model Factories

### (က) Model Factory ဖြင့် Dummy Data တည်ဆောက်ခြင်း
```bash
php artisan make:factory ProductFactory --model=Product
```

```php
// database/factories/ProductFactory.php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class ProductFactory extends Factory
{
    public function definition(): array
    {
        $name = fake()->words(3, true);
        return [
            'category_id' => 1,
            'title'       => ucfirst($name),
            'slug'        => Str::slug($name),
            'price'       => fake()->numberBetween(5000, 100000),
            'description' => fake()->paragraph(),
            'is_active'   => fake()->boolean(80), // 80% chance true
        ];
    }
}
```

### (ခ) DatabaseSeeder တွင် ခေါ်ယူခြင်း
```php
// database/seeders/DatabaseSeeder.php
public function run(): void
{
    // Admin User တစ်ယောက် မဖြစ်မနေ ထည့်မည်
    \App\Models\User::factory()->create([
        'name'  => 'Super Admin',
        'email' => 'admin@gmail.com',
        'role'  => 'admin',
    ]);

    // Dummy Products အခု ၅၀ ထည့်သွင်းမည်
    \App\Models\Product::factory(50)->create();
}
```

---

## ၇။ မကြာခဏသုံးရသော Artisan Database Commands

| Command | အဓိပ္ပာယ်နှင့် အသုံးပြုပုံ |
| :--- | :--- |
| `php artisan migrate` | မ run ရသေးသော migration ဖိုင်များကို execute လုပ်ခြင်း |
| `php artisan migrate:status` | မည်သည့် migration များ ပြီးဆုံးပြီး မည်သည့်အရာများ ကျန်ရှိနေသေးကြောင်း စစ်ဆေးခြင်း |
| `php artisan migrate:rollback` | နောက်ဆုံး run ခဲ့သော migration အသုတ် (batch) ကို ပြန်ဖျက်ခြင်း |
| `php artisan migrate:fresh` | Table အားလုံးကို Drop ချပြီး အစမှ အသစ်ပြန် run ခြင်း (သတိ: Data အားလုံး ပျက်မည်) |
| `php artisan migrate:fresh --seed` | Table အားလုံး ပြန်ဆောက်ပြီး Fake Seeder Data ပါ တစ်ခါတည်း သွင်းပေးခြင်း |
| `php artisan db:seed` | Seeder များကို သီးသန့် run ခြင်း |
