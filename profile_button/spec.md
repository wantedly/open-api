# 認証プロフィール追加ボタンの実装仕様

## SDK

Script タグを body タグの最後に入れてください。
この script はページ内で一回だけ読み込めば十分です。

```html
<script src="https://platform.wantedly.com/profile_buttons/script.js"></script>
```

## ボタンのプレースホルダ

ボタンを配置したいところにプレースホルダ要素(仮要素)を置きます。このプレースホルダはページ内に複数あっても構いません。

⚠️ この仮要素 `<a>` は SDK によって実際にユーザに表示されるボタン(`<iframe>`)に自動で置き換えられます。

```html
<a
  class="wantedly-profile-button"
  href="https://platform.wantedly.com/profile_buttons?PARAMS"
>Wantedlyに追加</a>
```

`?PARAMS` の部分は「URL パラメタ」を参照ください。

### サイズの変更

ボタンの大きさは `data-wtd-button-size` という data 属性によってコントロールできます。

```html
<a
  class="wantedly-profile-button"
  href="https://platform.wantedly.com/profile_buttons?PARAMS"
  data-wtd-button-size="240x44"
>Wantedlyに追加</a>
```

| 属性 | 形式 | 初期値 | 説明 |
|---|---|---|---|
| `data-wtd-button-size` | `{width}x{height}` | `240x44` | ボタンのサイズ |

⚠️ 指定できるのはボタン自体のサイズです。SDK によって置き換えられた `<iframe>` のサイズはガイドラインで規定されている isolation area を含むサイズになっています。

### Disabled (無効化状態)

「ボタンの利用に関するガイドライン」で明記されているように、コースの修了前(或いは受講完了前)にボタンをクリックして、Wantedly プロフィールに情報を追加することはできません。
そのため実際にボタンをクリックできるようになるまでは disabled な (無効化状態の) ボタンを表示しておく必要があります。

ボタンを無効化状態は、`href` 属性を指定しない (或いは空にする) ことによって実現可能です。

```html
<a
  class="wantedly-profile-button"
  data-wtd-button-size="240x44"
>Wantedlyに追加</a>
```

表示例:

![](https://user-images.githubusercontent.com/1695538/65211345-59270a80-dad9-11e9-92c2-2eb311eec2c6.png)


### URL パラメタ

ボタンの `href` に指定する URL はコース情報等を含む設定値を www-form-urlencoded 形式にし、URL パラメタとして `https://platform.wantedly.com/profile_buttons?` の末尾に追加したものになります。

ボタンの URL の例:

```
https://platform.wantedly.com/profile_buttons?partner_uuid=ec7a8fc4-6731-4467-ba11-c196ce02f0b9&course_uid=70377&user_uid=5286528&partner_name=%E3%82%A6%E3%82%A9%E3%83%B3%E3%83%86%E3%83%83%E3%83%89%E3%83%AA%E3%83%BC%E5%A4%A7%E5%AD%A6&course_name=%E3%82%B9%E3%83%BC%E3%83%91%E3%83%BC%E3%82%AF%E3%83%AA%E3%82%A8%E3%82%A4%E3%82%BF%E3%83%BC%E8%AC%9B%E5%BA%A7&course_url=https%3A%2F%2Fexample.com%2Fcourses%2F70377&required_learning_minutes=1500&certified=true&acquired_at=1568277853&course_type=certification&sign_mode=strict&signature=95912303d31c2eb61ddbcf0a5d921a302ecc72669b280c1b235d4780128a93b8
```

| キー | 型 | 必須・初期値 | 説明 |
|---|---|---|---|
| `partner_uuid` | 文字列 | 必須 | 提携プログラムの識別子。Wantedly が発行します。 |
| `course_uid` | 文字列 | 必須 | コースの識別子。提携プログラム内でコースを一意に特定している値を使います。 |
| `user_uid` | 文字列 | 必須 | 受講者の識別子。提携プログラム内でユーザを一意に特定している値を使います。 |
| `partner_name` | 文字列 | 必須 | 提携プログラムの名前。Wantedly プロフィール上で表示されます。 |
| `course_name` | 文字列 | 必須 | コースの名前。Wantedly プロフィール上で表示されます。 |
| `course_url` | 文字列 | 必須 | コースの説明ページの URL。プロフィールページからリンクされます。 |
| `required_learning_minutes` | 整数 | 必須 | コースの受講完了・修了に必要な時間 [単位: 分]。 |
| `certified` | ブール | `false` | 評価の完了。`true` = 評価済み、`false` = 評価なし・評価未完了。 |
| `acquired_at` | 整数 | 必須 | 受講完了あるいは認定取得日時のどちらか新しい方。[Epoch time](https://ja.wikipedia.org/wiki/UNIX時間) で表現します。 |
| `course_type` | 文字列 | 自動推定 | Wantedly プロフィール上での扱い。`skill` = スキル、`certification` = 資格・認定、`school` ＝ 学歴相当。<br>指定しない場合は `required_learning_minutes` と `certified` から推定されます。 |
| `sign_mode` | 文字列 | 必須 | 署名のモード。情報の改ざん・偽装防止のアルゴリズムの強度が設定できます。<br>`simple` = 簡易モード (非推奨): コースの情報のみ検証される。<br>`strict` = 厳格モード (推奨): コースと受講者のすべての情報が検証される。 |
| `signature` | 文字列 | 必須 | 署名の値。情報が改ざん・偽装されていないかを検証するために使われます。 |
| `debug` | ブール | `false` | 開発環境において、デバッグ画面を表示する場合は `true` を指定します。 |

## 署名の計算方法

署名の値は、署名のモードによって計算方法が変わります。<br>
どちらのモードを利用すべきかについては以下の比較表を参考にしてください。

| 署名のモード | 利点 | 欠点 | どういうケースに適しているか |
|---|---|---|---|
| `simple` (簡易モード) | • フォームから署名値が取得可能<br>• コースの名前は改ざん不可能 | • 受講者の情報は正確性を検証不可能<br>• コースの名前が変わった場合に対応が必要<br>• 新しいコースが増えた際は個別に対応が必要 | • コースの数があまり増減しない場合<br>• 改ざんによるリスクを許容できる場合<br>• サーバサイドに複雑なプログラムを入れられない場合 |
| `strict` (厳格モード) | • すべての情報の改ざん・偽装が不可能<br>• 新しいコースが増えた場合に対応が不要 | • サーバーサイドで署名値を計算する必要 | • コースの数が増減することが計画されている場合<br>• 改ざんによるリスクを取りたくない場合 |

### 簡易モードの場合

`sign_mode=simple` の場合は、以下の4つの値のみが改ざんできなくなります。

- `partner_uuid`
- `course_uid`
- `partner_name`
- `course_name`

こちらのモードには、署名の計算のために専用のフォームを用意しています。<br>
https://www.wantedly.com/platform_partners/new_signature にて、`secret_key` と上記4つの値をフォームに入力すると署名値が取得できます。

### 厳格モードの場合

`sign_mode=strict` の場合は、`sign_mode` と `signature` を除く全ての値が改ざんできなくなります。

- `partner_uuid`
- `course_uid`
- `user_uid`
- `partner_name`
- `course_name`
- `course_url`
- `required_learning_minutes`
- `certified`
- `acquired_at`
- `course_type`

この10個の値から署名を生成するには以下の手順に従います。

1. パラメタの値はすべて文字列表現にし、文字列のエンコーディングは UTF8 にします。
2. パラメタの値をキーの辞書順にし `:` (colon) によって結合した文字列を作ります。
3. 2\. で作った文字列をメッセージ(データ)として、これを [HMAC-SHA256](https://ja.wikipedia.org/wiki/HMAC) を用い、秘密鍵 `secret_key` で署名します。

<details><summary>Ruby での実装例 (外部ライブラリの依存なし)</summary>

```ruby
require "openssl"

# Wantedly から発行される情報
PARTNER_UUID = "ec7a8fc4-6731-4467-ba11-c196ce02f0b9"
SECRET_KEY = "9c61ef2e79993dccfbcdf998f93b7b836f7d9316b3ea2a48123e9cf87c5373d0"

# 設定値
params = {
  partner_uuid: PARTNER_UUID,
  course_uid: "70377",
  user_uid: "5286528",
  partner_name: "ウォンテッドリー大学",
  course_name: "スーパークリエイター講座",
  course_url: "https://example.com/courses/70377",
  required_learning_minutes: 1500,
  certified: true,
  acquired_at: 1568277853,
  course_type: "certification",
}

# 署名値の計算
verification_keys = params.keys.sort # キーの辞書順
sign_data = params.values_at(*verification_keys).join(':') # 辞書順で値を取得して : で結合
sign_key = [SECRET_KEY].pack('H*') # hex をデコード
signature = OpenSSL::HMAC.hexdigest("SHA256", sign_key, sign_data)

# パラメタに追加
params[:sign_mode] = 'strict'
params[:signature] = signature

# ボタンの URL
button_href = "https://platform.wantedly.com/profile_buttons?#{URI.encode_www_form(params)}"

printf "署名データ:\n%s\n\n", sign_data
printf "署名値:\n%s\n\n", signature
printf "ボタンのURL:\n%s\n\n", button_href
```

</details>

<details><summary>PHP での実装例 (外部ライブラリの依存なし)</summary>

```php
// Wantedly から発行される情報
const PARTNER_UUID = "ec7a8fc4-6731-4467-ba11-c196ce02f0b9";
const SECRET_KEY = "9c61ef2e79993dccfbcdf998f93b7b836f7d9316b3ea2a48123e9cf87c5373d0";

// 設定値
$params = array(
  "partner_uuid" => PARTNER_UUID,
  "course_uid" => "70377",
  "user_uid" => "5286528",
  "partner_name" => "ウォンテッドリー大学",
  "course_name" => "スーパークリエイター講座",
  "course_url" => "https://example.com/courses/70377",
  "required_learning_minutes" => 1500,
  "certified" => "true", // 注意: 文字列にしないと `1` になってしまう
  "acquired_at" => 1568277853,
  "course_type" => "certification",
);

// 署名値の計算
ksort($params, SORT_STRING); // 辞書順にソート
$sign_data = implode(":", array_values($params)); // 辞書順で値を取得して : で結合
$sign_key = hex2bin(SECRET_KEY); // hex をデコード
$signature = hash_hmac("sha256", $sign_data, $sign_key, false);

// パラメタに追加
$params["sign_mode"] = "strict";
$params["signature"] = $signature;

// ボタンの URL
$button_href = "https://platform.wantedly.com/profile_buttons?" . http_build_query($params);

printf("署名データ:\n%s\n\n", $sign_data);
printf("署名値:\n%s\n\n", $signature);
printf("ボタンのURL:\n%s\n\n", $button_href);
```

</details>

実行結果:

```
署名データ:
1568277853:true:スーパークリエイター講座:certification:70377:https://example.com/courses/70377:ウォンテッドリー大学:ec7a8fc4-6731-4467-ba11-c196ce02f0b9:1500:5286528

署名値:
95912303d31c2eb61ddbcf0a5d921a302ecc72669b280c1b235d4780128a93b8

ボタンのURL:
https://platform.wantedly.com/profile_buttons?partner_uuid=ec7a8fc4-6731-4467-ba11-c196ce02f0b9&course_uid=70377&user_uid=5286528&partner_name=%E3%82%A6%E3%82%A9%E3%83%B3%E3%83%86%E3%83%83%E3%83%89%E3%83%AA%E3%83%BC%E5%A4%A7%E5%AD%A6&course_name=%E3%82%B9%E3%83%BC%E3%83%91%E3%83%BC%E3%82%AF%E3%83%AA%E3%82%A8%E3%82%A4%E3%82%BF%E3%83%BC%E8%AC%9B%E5%BA%A7&course_url=https%3A%2F%2Fexample.com%2Fcourses%2F70377&required_learning_minutes=1500&certified=true&acquired_at=1568277853&course_type=certification&sign_mode=strict&signature=95912303d31c2eb61ddbcf0a5d921a302ecc72669b280c1b235d4780128a93b8
```

これをもとに生成されるプレースホルダーは以下のようになります。

```html
<a
  class="wantedly-profile-button"
  href="https://platform.wantedly.com/profile_buttons?partner_uuid=ec7a8fc4-6731-4467-ba11-c196ce02f0b9&course_uid=70377&user_uid=5286528&partner_name=%E3%82%A6%E3%82%A9%E3%83%B3%E3%83%86%E3%83%83%E3%83%89%E3%83%AA%E3%83%BC%E5%A4%A7%E5%AD%A6&course_name=%E3%82%B9%E3%83%BC%E3%83%91%E3%83%BC%E3%82%AF%E3%83%AA%E3%82%A8%E3%82%A4%E3%82%BF%E3%83%BC%E8%AC%9B%E5%BA%A7&course_url=https%3A%2F%2Fexample.com%2Fcourses%2F70377&required_learning_minutes=1500&certified=true&acquired_at=1568277853&course_type=certification&sign_mode=strict&signature=95912303d31c2eb61ddbcf0a5d921a302ecc72669b280c1b235d4780128a93b8"
>Wantedlyに追加</a>
```

## デバッグ画面

デバッグ画面に関して
正式リリース直前までは、プロフィール画面 (概要 図2) の代わりにデバッグ画面が表示されるようになっています。

**連携に成功した場合**

Verified: true が表示され、Errors の下に何もメッセージが出ていない状態になります。

![](https://user-images.githubusercontent.com/1695538/65676046-2efdbb80-e08a-11e9-8498-00edf0141099.png)

**エラーがある場合**

署名が不正な値の場合は Verified: false が表示されます。
その他の問題点は Errors の下に表示されます。

![](https://user-images.githubusercontent.com/1695538/65676048-2efdbb80-e08a-11e9-919f-b959f61fe6a3.png)
