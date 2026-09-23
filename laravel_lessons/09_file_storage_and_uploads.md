# 📁 သင်ခန်းစာ (၉) - File Storage နှင့် File Uploads စနစ်
### (Lesson 9: Laravel Filesystem, Public Disks, Storage Symlink & Cloud S3)

---

## 📌 မာတိကာ (Contents)
1. [Laravel Filesystem Abstraction ဆိုတာဘာလဲ?](#၁-laravel-filesystem-abstraction-ဆိုတာဘာလဲ)
2. [Storage Disks အမျိုးအစားများ (`local`, `public`, `s3`)](#၂-storage-disks-အမျိုးအစားများ)
3. [`php artisan storage:link` ဘာကြောင့် run ရသလဲ?](#၃-php-artisan-storagelink-ဘာကြောင့်-run-ရသလဲ)
4. [File Uploads ပြုလုပ်ခြင်းနှင့် လုံခြုံရေး Validation စစ်ဆေးခြင်း](#၄-file-uploads-ပြုလုပ်ခြင်းနှင့်-လုံခြုံရေး-validation)
5. [ဖိုင်များ ဖတ်ယူခြင်း၊ ပြသခြင်းနှင့် ဖျက်ပစ်ခြင်း](#၅-ဖိုင်များ-ဖတ်ယူခြင်း-ပြသခြင်းနှင့်-ဖျက်ပစ်ခြင်း)
6. [Profile Avatar ပြင်ဆင်ခြင်း လက်တွေ့ Code နမူနာ (Old File Auto Delete)](#၆-profile-avatar-ပြင်ဆင်ခြင်း-လက်တွေ့-code-နမူနာ)

---

## ၁။ Laravel Filesystem Abstraction ဆိုတာဘာလဲ?

Laravel သည် **Flysystem** PHP Library ကို အသုံးပြုထားသောကြောင့် File စီမံခန့်ခွဲမှု (ဖိုင်သိမ်းခြင်း၊ ဖတ်ခြင်း၊ ဖျက်ခြင်း) ကို Disk အမျိုးအစား မည်သို့ပင်ဖြစ်စေ ကုဒ်တစ်မျိုးတည်းဖြင့် လွယ်ကူစွာ လုပ်ဆောင်နိုင်စေသည်။

သင်၏ Code တွင် Local Hard Drive ပေါ်သို့ သိမ်းသည်ဖြစ်စေ၊ Amazon S3 Cloud ပေါ်သို့ သိမ်းသည်ဖြစ်စေ Syntax ပြောင်းလဲရန် မလိုဘဲ `.env` ဖိုင်မှ Driver ပြောင်းရုံဖြင့် အလုပ်ဖြစ်စေသည်။

---

## ၂။ Storage Disks အမျိုးအစားများ

`config/filesystems.php` တွင် Disks များကို စီမံထားပါသည်:
* **`local` Disk** (`storage/app/private`): ပြင်ပ Web Browser မှ တိုက်ရိုက် ကြည့်ခွင့်မပေးလိုသော လျှို့ဝှက်ဖိုင်များ (ဥပမာ- Invoices, Contracts, Backup files) သိမ်းရန်။
* **`public` Disk** (`storage/app/public`): အများပြည်သူ ကြည့်ရှုနိုင်သော ဖိုင်များ (ဥပမာ- Profile avatars, Product photos, Blog covers) သိမ်းရန်။
* **`s3` Disk** (Amazon Web Services S3): Server disk ပြည့်မသွားစေရန်နှင့် Production စနစ်ကြီးများအတွက် Cloud ပေါ်သို့ ဖိုင်များ သိမ်းရန်။

---

## ၃။ `php artisan storage:link` ဘာကြောင့် run ရသလဲ?

Web Server (Nginx/Apache) ၏ Document Root သည် `public/` directory သာ ဖြစ်သောကြောင့် `storage/` directory ထဲရှိ ဖိုင်များကို Browser က တိုက်ရိုက် မလှမ်းနိုင်ပါ။

ထို့ကြောင့် `storage/app/public` folder ကို `public/storage` အဖြစ် ချိတ်ဆက်ပေးသော **Symbolic Link (Shortcut Link)** ကို ဖန်တီးပေးရပါသည်:

```bash
php artisan storage:link
```
ဤ Command ကို Run ပြီးသည်နှင့် `public/storage/...` မှတစ်ဆင့် ပုံများကို Browser တွင် ခေါ်ယူပြသနိုင်သွားမည် ဖြစ်သည်။

---

## ၄။ File Uploads ပြုလုပ်ခြင်းနှင့် လုံခြုံရေး Validation

### (က) HTML Form ပြင်ဆင်ခြင်း
Form တွင် `enctype="multipart/form-data"` မဖြစ်မနေ ပါဝင်ရမည်:
```html
<form action="{{ route('avatar.upload') }}" method="POST" enctype="multipart/form-data">
    @csrf
    <input type="file" name="avatar" accept="image/*">
    <button type="submit">Upload တင်မည်</button>
</form>
```

### (ခ) Validation စစ်ဆေးခြင်း
File တင်သည့်အခါ Hacker များ malicious script (PHP shell) များ မတင်နိုင်စေရန် တင်းကြပ်စွာ စစ်ဆေးရပါသည်:
```php
$request->validate([
    'avatar' => [
        'required',
        'image',                           // Image ဖိုင် အစစ် ဟုတ်/မဟုတ် စစ်ဆေးသည်
        'mimes:jpeg,png,jpg,webp',         // File extension ကန့်သတ်ခြင်း
        'max:2048',                        // 2MB (2048 KB) ထက် မကျော်ရ
        'dimensions:min_width=100,min_height=100' // ပုံအရွယ်အစား အနည်းဆုံး စစ်ဆေးခြင်း
    ],
]);
```

---

## ၅။ ဖိုင်များ ဖတ်ယူခြင်း၊ ပြသခြင်းနှင့် ဖျက်ပစ်ခြင်း

```php
use Illuminate\Support\Facades\Storage;

// ၁။ ဖိုင်ကို အလိုအလျောက် Unique Hash အမည်ပေး၍ သိမ်းခြင်း
// ရလဒ်: 'avatars/xY7zAbc123.jpg'
$path = $request->file('avatar')->store('avatars', 'public');

// ၂။ မိမိစိတ်ကြိုက် အမည်သတ်မှတ်၍ သိမ်းခြင်း
$customName = 'user_' . auth()->id() . '.' . $request->file('avatar')->getClientOriginalExtension();
$path = $request->file('avatar')->storeAs('avatars', $customName, 'public');

// ၃။ ဖိုင် ရှိမရှိ စစ်ဆေးခြင်း
if (Storage::disk('public')->exists($path)) {
    // ဖိုင် တည်ရှိသည်
}

// ၄။ ဖိုင်ကို ပြန်ဖျက်ခြင်း
Storage::disk('public')->delete($path);

// ၅။ Public URL ရယူခြင်း
$url = Storage::url($path); // /storage/avatars/xY7zAbc123.jpg
```

---

## ၆။ Profile Avatar ပြင်ဆင်ခြင်း လက်တွေ့ Code နမူနာ

အသုံးပြုသူ ပုံဟောင်းရှိပြီး အသစ်ထပ်တင်ပါက ပုံဟောင်းကို Server ပေါ်မှ အလိုအလျောက် ဖျက်ပေးမည့် စနစ်:

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class ProfileController extends Controller
{
    public function updateAvatar(Request $request)
    {
        $request->validate([
            'avatar' => 'required|image|mimes:jpeg,png,webp|max:2048',
        ]);

        $user = $request->user();

        if ($request->hasFile('avatar')) {
            // ၁။ ယခင် ပုံဟောင်း ရှိပါက Server ပေါ်မှ အရင်ဖျက်ပစ်မည်
            if ($user->avatar && Storage::disk('public')->exists($user->avatar)) {
                Storage::disk('public')->delete($user->avatar);
            }

            // ၂။ ပုံအသစ်ကို public storage သို့ သိမ်းမည်
            $path = $request->file('avatar')->store('avatars', 'public');

            // ၃။ Database ထဲတွင် Path အသစ်ကို update လုပ်မည်
            $user->update([
                'avatar' => $path
            ]);
        }

        return back()->with('success', 'Profile ပုံ အောင်မြင်စွာ ပြောင်းလဲပြီးပါပြီ!');
    }
}
```

#### Blade View တွင် ပုံထုတ်ပြပုံ:
```html
@if(auth()->user()->avatar)
    <img src="{{ asset('storage/' . auth()->user()->avatar) }}" alt="Avatar" width="120" height="120" class="rounded-full">
@else
    <img src="{{ asset('images/default-avatar.png') }}" alt="Default Avatar" width="120" height="120">
@endif
```
