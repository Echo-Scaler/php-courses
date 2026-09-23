# 🎨 သင်ခန်းစာ (၄) - Blade Template Engine (Frontend UI တည်ဆောက်ခြင်း)
### (Lesson 4: Master Blade Templating, Layouts, Components & Directives)

---

## 📌 မာတိကာ (Contents)
1. [Blade Template Engine ဆိုတာဘာလဲ?](#၁-blade-template-engine-ဆိုတာဘာလဲ)
2. [Layout Inheritance (`@extends`, `@yield`, `@section`)](#၂-layout-inheritance-extends-yield-section)
3. [Modern Blade Components နှင့် Slots (`<x-component>`)](#၃-modern-blade-components-နှင့်-slots)
4. [မဖြစ်မနေ သိထားရမည့် Blade Directives များ](#၄-မဖြစ်မနေ-သိထားရမည့်-blade-directives-များ)
5. [The `$loop` Variable ၏ အစွမ်း](#၅-the-loop-variable-၏-အစွမ်း)
6. [Data Displaying & XSS Security (`{{ }}` vs `{!! !!}`)](#၆-data-displaying--xss-security)
7. [လက်တွေ့ Dashboard Layout တည်ဆောက်ပုံ နမူနာ](#၇-လက်တွေ့-dashboard-layout-တည်ဆောက်ပုံ-နမူနာ)

---

## ၁။ Blade Template Engine ဆိုတာဘာလဲ?

**Blade** သည် Laravel Framework တွင် မူလအသင့်ပါဝင်သော ရိုးရှင်းပြီး အလွန်မြန်ဆန်သည့် **Templating Engine** ဖြစ်သည်။
* ဖိုင်အမည်များကို `.blade.php` ဖြင့် အဆုံးသတ်ရသည်။
* Blade ကုဒ်များသည် Plain PHP အဖြစ် compile လုပ်ကာ Cache သိမ်းထားသောကြောင့် စွမ်းဆောင်ရည် အလွန်မြင့်မားသည်။
* View ဖိုင်များအားလုံးကို `resources/views/` folder အောက်တွင် သိမ်းဆည်းရသည်။

Controller မှ View ဖိုင်ကို ခေါ်ယူပုံ:
```php
// resources/views/products/index.blade.php ကို ခေါ်ယူခြင်း
return view('products.index', [
    'title' => 'ကုန်ပစ္စည်းများ',
    'items' => $products
]);
```

---

## ၂။ Layout Inheritance (`@extends`, `@yield`, `@section`)

Web Application တစ်ခုတွင် Navigation Bar, Sidebar, Footer စသည်တို့သည် စာမျက်နှာတိုင်းတွင် တူညီနေလေ့ရှိသည်။ Layout Inheritance ဖြင့် Parent Template တစ်ခုတည်း ပြုလုပ်ထားပြီး Child Pages များတွင် ပြန်လည်အသုံးပြုနိုင်သည်။

### (က) Master Layout (`resources/views/layouts/master.blade.php`):
```html
<!DOCTYPE html>
<html lang="my">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'My Application')</title>
    <!-- Stylesheets -->
    <link rel="stylesheet" href="{{ asset('css/app.css') }}">
    @stack('styles')
</head>
<body>
    <header>
        <nav>
            <a href="/">Home</a>
            <a href="/products">Products</a>
            <a href="/about">About</a>
        </nav>
    </header>

    <main class="container">
        {{-- Flash Success Message --}}
        @if(session('success'))
            <div class="alert alert-success">{{ session('success') }}</div>
        @endif

        {{-- Child View ၏ အဓိက Content ထည့်သွင်းမည့်နေရာ --}}
        @yield('content')
    </main>

    <footer>
        <p>&copy; {{ date('Y') }} Myanmar Web Academy. All rights reserved.</p>
    </footer>

    @stack('scripts')
</body>
</html>
```

### (ခ) Child View (`resources/views/about.blade.php`):
```html
@extends('layouts.master')

@section('title', 'ကျွန်ုပ်တို့အကြောင်း (About Us)')

@section('content')
    <h1>About Our Company</h1>
    <p>Laravel အခြေခံမှ စတင်ပြီး ကျွမ်းကျင်သူအဆင့်ထိ လေ့လာနိုင်သော နေရာဖြစ်ပါသည်။</p>
@endsection
```

---

## ၃။ Modern Blade Components နှင့် Slots

Laravel တွင် UI Elements များကို ပြန်လည်အသုံးပြုနိုင်သော Components များအဖြစ် သီးသန့်ခွဲထုတ်ရေးသားနိုင်သည်:

```bash
# Component အသစ် ဖန်တီးခြင်း
php artisan make:component Alert
```

### (က) Alert Component (`resources/views/components/alert.blade.php`):
```html
@props(['type' => 'info', 'message'])

<div class="alert alert-{{ $type }}">
    <strong>{{ ucfirst($type) }}:</strong>
    {{ $message ?? $slot }}
</div>
```

### (ခ) အခြား View များတွင် Component အား အသုံးပြုပုံ:
```html
{{-- Property ဖြင့် ပို့ခြင်း --}}
<x-alert type="danger" message="အချက်အလက် ထည့်သွင်းမှု မှားယွင်းနေပါသည်။" />

{{-- Slot Body ဖြင့် ပို့ခြင်း --}}
<x-alert type="success">
    သင်၏ အကောင့်ဖွင့်ခြင်း <strong>အောင်မြင်ပါသည်!</strong>
</x-alert>
```

---

## ၄။ မဖြစ်မနေ သိထားရမည့် Blade Directives များ

### (က) Conditionals (အခြေအနေ စစ်ဆေးခြင်း)
```html
@if($user->role === 'admin')
    <p>မင်္ဂလာပါ Admin!</p>
@elseif($user->role === 'editor')
    <p>မင်္ဂလာပါ စာတည်း!</p>
@else
    <p>မင်္ဂလာပါ အသုံးပြုသူ!</p>
@endif

{{-- Unless (If not နှင့် အတူတူဖြစ်သည်) --}}
@unless(Auth::check())
    <a href="/login">Login ဝင်ရောက်ပါ</a>
@endunless

{{-- Login ဝင်ထားသလား စစ်ဆေးခြင်း --}}
@auth
    <p>Login အသုံးပြုသူ: {{ auth()->user()->name }}</p>
@endauth

@guest
    <a href="/login">Sign In</a>
@endguest
```

### (ခ) Loops (ထပ်ခါတလဲလဲ ပတ်ခြင်း)
```html
{{-- @forelse: ဒေတာရှိရင် ပတ်ပြီး မရှိရင် empty ပြပေးသည် --}}
<ul>
@forelse($products as $product)
    <li>{{ $product->name }} - {{ number_format($product->price) }} MMK</li>
@empty
    <li>လက်ရှိ ရောင်းချမည့် ပစ္စည်း မရှိသေးပါ။</li>
@endforelse
</ul>
```

### (ဂ) Form Security Directives
```html
<form action="/products/1" method="POST">
    {{-- CSRF Token မဖြစ်မနေ ထည့်ရပါမည် --}}
    @csrf

    {{-- HTML Form တွင် PUT method သုံးရန် --}}
    @method('PUT')

    <input type="text" name="name" value="{{ old('name', $product->name) }}">

    {{-- Error ရှိမရှိ စစ်ဆေးပြီး ပြသခြင်း --}}
    @error('name')
        <span class="text-danger">{{ $message }}</span>
    @enderror

    <button type="submit">ပြင်ဆင်မည်</button>
</form>
```

---

## ၅။ The `$loop` Variable ၏ အစွမ်း

`@foreach` ပတ်တိုင်း Laravel က `$loop` object ကို အလိုအလျောက် ထည့်ပေးထားသည်:

```html
@foreach($users as $user)
    <div class="{{ $loop->even ? 'bg-gray-100' : 'bg-white' }}">
        #{{ $loop->iteration }} - {{ $user->name }}
        
        @if($loop->first)
            <span class="badge">ပထမဆုံး Record</span>
        @endif

        @if($loop->last)
            <span class="badge">နောက်ဆုံး Record</span>
        @endif
    </div>
@endforeach
```
* `$loop->iteration` : 1 မှ စတင်ရေတွက်သော Current Index။
* `$loop->first` : ပထမဆုံး အကြိမ်ဖြစ်ပါက `true`။
* `$loop->last` : နောက်ဆုံး အကြိမ်ဖြစ်ပါက `true`။
* `$loop->count` : စုစုပေါင်း item အရေအတွက်။

---

## ၆။ Data Displaying & XSS Security

```html
{{-- ၁။ Escaped Output (Default - အလွန် လုံခြုံသည်) --}}
{{-- Browser က Script tags များကို Run မသွားအောင် htmlspecialchars ဖြင့် escape လုပ်ပေးသည် --}}
<p>{{ $userInput }}</p>

{{-- ၂။ Raw Output (Unescaped - HTML ကို run စေသည်) --}}
{{-- သတိပြုရန်: အသုံးပြုသူထံမှ လာသော Data ကို ဤပုံစံဖြင့် မပြရပါ။ (XSS Attack ဖြစ်နိုင်သည်) --}}
<div>{!! $trustedArticleHtmlContent !!}</div>

{{-- ၃။ JavaScript ထဲသို့ Data ပို့ခြင်း --}}
<script>
    const user = {{ Js::from($user) }};
    console.log(user.name);
</script>
```

---

## ၇။ လက်တွေ့ Dashboard Layout တည်ဆောက်ပုံ နမူနာ

```html
<!-- resources/views/dashboard.blade.php -->
@extends('layouts.master')

@section('title', 'Admin Dashboard')

@section('content')
<div class="row">
    <div class="col-md-4">
        <div class="card">
            <h3>စုစုပေါင်း အသုံးပြုသူ</h3>
            <p class="display-4">{{ $totalUsers }}</p>
        </div>
    </div>
    <div class="col-md-4">
        <div class="card">
            <h3>ယနေ့ ရောင်းရငွေ</h3>
            <p class="display-4">{{ number_format($todayRevenue) }} MMK</p>
        </div>
    </div>
</div>
@endsection
```
