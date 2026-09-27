# 雨音：Claude Code 向けメモ

## このリポジトリについて

- 雨音を聴く Web アプリです。`index.html` の1ファイルだけで動きます（ビルド不要、依存パッケージなし）
- 外部から読み込むのは Google Fonts だけです
- 公開先は GitHub Pages、告知先は X です

## 今回お願いしたい作業

1. **ユーザーに確認する**
   - リポジトリ名（`amaoto` を想定）
   - 公開範囲（Public を想定）
   - ライセンス（未定。付けるかどうかと、付ける場合の種類）
2. **ユーザー名を置き換える**
   - `gh api user --jq .login` で GitHub のユーザー名を取得する
   - `index.html` と `README.md` の `USERNAME` をすべて、そのユーザー名に置き換える
   - OGP 画像の URL は絶対 URL でないと X にプレビューが出ないため、この置き換えは必須
   - リポジトリ名を `amaoto` 以外にする場合は、URL 内の `/amaoto/` も合わせて置き換える
3. **リポジトリを作ってアップロードする**
   - `git init` → `add` → `commit` の順に進める
   - `gh repo create <名前> --public --source . --push` で作成とプッシュをまとめて行う
4. **GitHub Pages を有効にする**
   - 公開元は main ブランチのルート（`/`）にする
   - コマンド例：`gh api -X POST repos/<owner>/<repo>/pages -f "source[branch]=main" -f "source[path]=/"`
5. **公開されたか確認する**
   - `https://<user>.github.io/<repo>/` と `.../og.png` がそれぞれ 200 を返すか確かめる
   - 反映には数分かかることがある
6. **ユーザーに伝える**
   - 公開 URL
   - X の投稿文の案（下の例を参考に）

### X 投稿文の例

```
雨が当たる音を聴くWebアプリ「雨音」を作りました☔
番傘、トタン屋根、瓦、窓ガラス、テント…12か所 × 雨の強さ（mm/h）を選べます。
音は録音ではなく、ブラウザの中でひと粒ずつ合成しています。
https://<user>.github.io/<repo>/
```

## 音作りで決まっていること（修正するときに守ること）

ユーザーが実際に聴いて指摘した内容です。頼まれていない限り、音の方向性は変えないでください。

- **「ピチョン」という音は入れない。**
  - 音程が上がっていく水滴の音（`addMode` の `rise` を使うもの）は使わない
  - しずくは音程のない「ばしゃ」「ぼと」にする
- **甲高く細かい音にしない。**
  - トタン屋根とビニール傘は、低めの「ボツボツ」が中心
  - 金属の響きは中音域で短くする
- **ガルバリウムは静かでこもった音にする。**
  - 野地板と断熱材越しに室内で聴く想定
  - 一粒ずつの音は目立たせない
- 室外機と車の中は、一度作ったうえで不要と判断された。再追加しない

## コードの構成（index.html 内）

| 名前 | 中身 |
|---|---|
| `SURF` | 場所ごとの設定（分類、名前、聴く位置、説明、粒の頻度、音量、フィルターなど） |
| `GROUPS` | 分類とその並び順 |
| `GEN` | ひと粒の音の作り方（場所ごと、しずく用） |
| `makeVoice` / `tick` / `drop` | 音の組み立てと、先読みで雨粒を鳴らすスケジューラ |
| `layout` / `layoutExtra` / `hitY` / `draw*` | Canvas に描く情景と、雨粒が当たる位置の判定 |
| `RAIN` / `RADAR` | 雨の強さの呼び方と色帯 |

場所を追加するときは、`SURF`・`GEN`・情景の描画（`layoutExtra` と `draw*`）の3か所をそろえて変更してください。

## OGP 画像（og.png）の作り直し

`tools/og.html` を 1200×630 で撮影したものが `og.png` です。乱数のシードを固定しているので、同じ HTML からは同じ絵になります。
Windows では Edge のヘッドレスモードで撮影します（Web フォントの読み込みを待つため `--virtual-time-budget` を付ける）。

```powershell
& "${env:ProgramFiles(x86)}\Microsoft\Edge\Application\msedge.exe" --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=1 --window-size=1200,630 --virtual-time-budget=8000 "--user-data-dir=$env:TEMP\og-prof" "--screenshot=$PWD\og.png" "file:///$($PWD -replace '\\','/')/tools/og.html"
```

撮影後は画像を目で見て、文字の切れ・フォントの反映を確認してから commit してください。

- 画像を差し替えたら、`index.html` の `og:image` と `twitter:image` の `?v=` の数字も 1 つ上げる。X は画像の URL ごとにキャッシュするため、URL が同じだと古い画像が出続ける
- 左下は X のカードでドメイン名が重なるので、文字を置かない
