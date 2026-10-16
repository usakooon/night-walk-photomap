# Night Walk 休憩所フォトマップ

青山学院大学 青山キャンパスから相模原キャンパスまでのNight Walkについて、実際に歩いたGPS軌跡と、休憩所の位置・写真を地図上で振り返るWebページです。

> この地図は撮影記録とYAMAPのGPS記録をもとに独自に作成したものです。

## ファイル構成

```
nightwalk/
├── index.html                          # 地図ページ本体
├── data/
│   ├── spots.json                      # 休憩所データ（ここを編集する）
│   └── yamap_2026-10-10_20_26.gpx      # 歩行軌跡（YAMAPの元ファイル。編集しない）
├── images/                             # 写真（表示用に軽量化したもの）を置く
└── README.md
```

すべて相対パスで読み込むので、GitHub Pagesのリポジトリ配下でもそのまま動きます。

---

## ローカルで確認する

`index.html` をダブルクリックで開いても、ブラウザの制限でデータを読み込めず動きません。
次のように簡易サーバーを立てて確認します（PowerShell、`nightwalk` フォルダの中で）。

```powershell
py -m http.server 8000
```

ブラウザで http://localhost:8000/ を開きます。止めるときは PowerShell で `Ctrl + C`。
`py` が使えないときは `python -m http.server 8000` を試してください。

---

## 歩行軌跡（GPX）

- 元のGPXは `data/` に置いてあり、**変更していません**。ページは読み込んで表示するだけです。
- GPXの時刻はUTCで記録されています。ページ上では**日本時間（JST）に変換**して表示します。
- 連続する2点が150 m以上離れている箇所は、**「記録の空白」**として破線で描き、実線（実際に歩いた道筋）と区別します。破線をタップすると、空白の時間帯が表示されます。
- 別のGPXに差し替えるときは、ファイルを `data/` に置き、`spots.json` の `meta.gpx` のパスを書き換えます。

```json
"meta": {
  "gpx": "data/yamap_2026-10-10_20_26.gpx",
  "reviewThresholdMeters": 100
}
```

## 「位置要確認」の表示について

休憩所の座標は施設の代表位置で、実際に休憩した場所とずれていることがあります。
そのため、**軌跡（記録の空白を除く、実際に歩いた線）から `reviewThresholdMeters` m 以上離れている休憩所には「位置要確認」と表示**します。

- **座標は自動では動きません。** 表示するだけです。
- 確認して座標を直したら、その休憩所の `lat` / `lng` を書き換え、`positionType` を実情に合わせます（下の表）。
- 表示が不要なら `"reviewThresholdMeters": null` にします。数値を小さくするとより厳しく判定します。

---

## 休憩所・写真の追加方法

### 1. 写真を `images/` に入れる

表示用に軽量化（目安: 長辺1600px、1枚1MB以下）した写真を入れます。**元の画像は変更せず、別に保管してください。**

```
images/
├── spot02_01.jpg   # 二子玉川公園 1枚目
├── spot02_02.jpg   # 二子玉川公園 2枚目
```

### 2. `data/spots.json` の `photos` に追記する

同じ休憩所に写真を何枚でも登録できます。

```json
{
  "id": 2,
  "name": "二子玉川公園",
  "lat": 35.608250,
  "lng": 139.632278,
  "positionType": "representative",
  "time": "00:40",
  "photoMethod": "fotoshi",
  "description": "ここで休憩した。",
  "photos": [
    { "src": "images/spot02_01.jpg", "caption": "入口付近", "time": "00:42" },
    { "src": "images/spot02_02.jpg", "caption": "ベンチ" }
  ]
}
```

| フィールド | 説明 |
|-----------|------|
| `positionType` | `representative`＝施設の代表位置 / `manual`＝手動で設定した位置 / `photo-gps`＝写真のGPS位置 / `actual`＝実測位置 |
| `photoMethod` | `fotoshi` / `camera`（通常のカメラ）/ `unknown`（不明）。写真がある地点で表示されます |
| `time` | 休憩所の時刻（日本時間の文字列。変換はされません） |
| `photos[].src` | 写真のパス（`images/...`） |
| `photos[].caption` / `time` | 写真の説明・撮影時刻（任意） |

位置が確定できない地点は、推測で座標を決めず、`positionType` を `representative` のまま残して確認します。
`spots.json` の書式が崩れると何も表示されません。編集後は https://jsonlint.com などで確認してください。

---

## GitHub Pagesで公開する

### 初回

```powershell
git init
git add .
git commit -m "Night Walk フォトマップ"
git branch -M main
git remote add origin https://github.com/アカウント名/リポジトリ名.git
git push -u origin main
```

GitHubのリポジトリで **Settings → Pages** を開き、**Branch** を `main` の `/ (root)` にして **Save** します。
数分後に `https://アカウント名.github.io/リポジトリ名/` で公開されます。

### 更新するとき

```powershell
git add .
git commit -m "写真を追加"
git push
```

---

## 地図の出典

- 地図データ: © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors
- 背景地図タイル: [OpenStreetMap](https://www.openstreetmap.org/)（標準タイル）
- 地図ライブラリ: [Leaflet](https://leafletjs.com/)
