# Laravelでファイルアップロードが機能しない問題の解決方法

Laravel Breezeでユーザー登録フォームにプロフィール画像を追加しようとした際に、ファイルアップロードが正しく動作しなかった問題について書きます。  
この問題を解決するために行ったデバッグの過程と解決策を共有します。

## 環境
- Laravel 11
- WSL2 Ubuntu 24.04
- Node.js v20

## 前提
- Laravel Breezeがインストールされていること
- Register に 画像を添付し送信できる仕様になっていること

## ファイルがリクエストに含まれない問題
`register.blade.php` にプロフィール画像のアップロード機能を追加し、フォームを送信したところ、ファイルがサーバー側に送信されていないことに気付きました。  
具体的には、dd($request->file($request)) でリクエストを確認した際に、ファイルがリクエストに含まれていませんでした。

### 原因と解決策
この問題の主な原因は、フォームタグに `enctype="multipart/form-data"` を指定していなかったことです。  
Laravelに限らず、ファイルをアップロードするHTMLフォームでは、この属性を指定する必要があります。これがないと、ファイルがリクエストに含まれません。

### 修正内容
register.blade.php の修正
以下のように、<form> タグに `enctype="multipart/form-data"` を追加しました。

```php
<form method="POST" action="{{ route('register') }}" enctype="multipart/form-data">
    @csrf

    <!-- ユーザー名 -->
    <div>
        <x-input-label for="username" :value="__('Username')" />
        <x-text-input id="username" class="block mt-1 w-full" type="text" name="username" :value="old('username')" required autofocus autocomplete="username" />
        <x-input-error :messages="$errors->get('username')" class="mt-2" />
    </div>

    <!-- メールアドレス -->
    <div class="mt-4">
        <x-input-label for="email" :value="__('Email')" />
        <x-text-input id="email" class="block mt-1 w-full" type="email" name="email" :value="old('email')" required autocomplete="username" />
        <x-input-error :messages="$errors->get('email')" class="mt-2" />
    </div>

    <!-- 説明 -->
    <div class="mt-4">
        <x-input-label for="description" :value="__('Description')" />
        <textarea id="description" class="block mt-1 w-full" name="description" rows="4" required placeholder="{{ __('Describe yourself here.') }}" maxlength="2000">{{ old('description') }}</textarea>
        <x-input-error :messages="$errors->get('description')" class="mt-2" />
    </div>

    <!-- プロフィール画像 -->
    <div class="mt-4">
        <x-input-label for="profile_picture" :value="__('Profile Picture')" />
        <input id="profile_picture" class="block mt-1 w-full" type="file" name="profile_picture" accept="image/*" required />
        <x-input-error :messages="$errors->get('profile_picture')" class="mt-2" />
    </div>

    <!-- パスワード -->
    <div class="mt-4">
        <x-input-label for="password" :value="__('Password')" />
        <x-text-input id="password" class="block mt-1 w-full" type="password" name="password" required autocomplete="new-password" />
        <x-input-error :messages="$errors->get('password')" class="mt-2" />
    </div>

    <!-- 確認用パスワード -->
    <div class="mt-4">
        <x-input-label for="password_confirmation" :value="__('Confirm Password')" />
        <x-text-input id="password_confirmation" class="block mt-1 w-full" type="password" name="password_confirmation" required autocomplete="new-password" />
        <x-input-error :messages="$errors->get('password_confirmation')" class="mt-2" />
    </div>

    <div class="flex items-center justify-end mt-4">
        <a class="underline text-sm text-gray-600 hover:text-gray-900 rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500" href="{{ route('login') }}">
            {{ __('Already registered?') }}
        </a>

        <x-primary-button class="ms-4">
            {{ __('Register') }}
        </x-primary-button>
    </div>
</form>
```

## ファイルの拡張子が空
ファイルが送信された後、次に発生した問題は、ファイルの拡張子が正しく取得されないことでした。dd($request->file('profile_picture')) で確認したところ、拡張子が空のままでした。これは、ファイル名の生成や保存処理に影響を与える可能性があります。

### 解決策：getClientOriginalExtension() の使用
アップロードされたファイルの拡張子を正しく取得するために、getClientOriginalExtension() を使用しました。このメソッドを使用することで、ファイルの元の拡張子を正確に取得できます。

### RegisteredUserController.php の修正
```php
public function store(Request $request): RedirectResponse
{
    // バリデーション
    $request->validate([
        'username' => ['required', 'string', 'max:255'],
        'email' => ['required', 'string', 'lowercase', 'email', 'max:255', 'unique:' . User::class],
        'description' => ['required', 'string', 'max:2000'],
        'profile_picture' => ['required', 'image', 'mimes:jpeg,png,jpg,webp', 'max:10240'],
        'password' => ['required', 'confirmed', Rules\Password::defaults()],
    ]);

    // アップロードされたファイルの取得
    $file = $request->file('profile_picture');

    // 拡張子の取得
    $extension = $file->getClientOriginalExtension();

    // ファイル名の生成
    $md5Filename = md5($file->getClientOriginalName() . $request->username . Carbon::now()->toDateString()) . '.' . $extension;

    // ファイルの保存
    $profilePicturePath = $file->storeAs('users/profiles/profile_pictures', $md5Filename, 'public');

    // ユーザーの作成
    $user = User::create([
        'username' => $request->username,
        'email' => $request->email,
        'description' => $request->description,
        'profile_path' => $profilePicturePath, // プロフィール画像の保存パスを保存
        'password' => Hash::make($request->password),
    ]);

    // 自動ログインとリダイレクト
    event(new Registered($user));
    Auth::login($user);

    return redirect(route('dashboard', absolute: false));
}
```

