# 燃料帳（fuel-ledger）

**English:** Fuel Ledger is a personal nutrition log for endurance athletes that runs as a private claude.ai Artifact. Snap a photo of your meal, Claude estimates calories and macros, and the result is appended to your own Google Sheet. Strava activities and (optionally) Withings weigh-ins are combined into a daily energy balance, energy availability, and weight trend. Because Artifacts that use connectors cannot be shared, each person publishes their own copy with the included setup skill. The UI and docs are in Japanese.

---

食事の写真とメモからClaudeが栄養を推定して、自分のGoogleスプレッドシートに記録するページです。Stravaの運動とWithingsの体重を合わせて、1日のエネルギー収支、エネルギー利用可能量、体重の推移を表示します。iPhoneのSafariで使う前提で作っています。

## できること

<!-- スクリーンショットをここに追加 -->
<!-- ![この日の収支](docs/images/day.png) -->
<!-- ![食事を記録](docs/images/add.png) -->

- 写真（複数可）とメモから、Claudeがkcal・たんぱく質・脂質・炭水化物を推定。確かめて直してからシートに1行追加
- 「摂取 − 消費 ＝ 収支」と、たんぱく質・脂質・炭水化物の目標までの残り
- Stravaの運動を、安静時の分を差し引いた「上乗せ」に換算して消費に足す
- エネルギー利用可能量（摂取 − 運動）が下限を割っていないかの表示
- 14日間の収支グラフ
- Withingsの体重の7日平均と、週ごとの体脂肪量（任意）

## 必要なもの

- **Claudeの有料プラン**（Pro／Max／Team／Enterprise）。写真からの推定は**自分のClaudeの使用量を使います**
- **Strava**の有料プラン（claude.aiのStravaコネクタに必要）
- **Googleスプレッドシート**（Googleアカウント）
- **Claude Code** か **Claudeデスクトップアプリ**（公開するときだけ）
- Withingsの体重計（任意）

## セットアップ

### ① テンプレートをコピーする

[テンプレートをコピー](https://docs.google.com/spreadsheets/d/TEMPLATE_SPREADSHEET_ID/copy) を開き、「コピーを作成」を押します。「食事ログ」「設定」の2つのタブがあるシートが自分のドライブにできます。

シートを自分で作る場合は [template/README.md](template/README.md) の列定義のとおりに作ってください。

### ② claude.aiでGoogle SheetsとStravaを連携する

claude.aiの **設定 → コネクタ** で、**Google Sheets** と **Strava** を追加してサインインします。

### ③ スキルを入れて「燃料帳をセットアップして」と頼む

1. このリポジトリを手元に置きます。

   ```bash
   git clone https://github.com/hirokitnk/fuel-ledger.git
   ```

2. スキルを入れます（Claude Codeの場合）。

   ```bash
   mkdir -p ~/.claude/skills && cp -r fuel-ledger/skills/fuel-ledger-setup ~/.claude/skills/
   ```

3. Claude Codeかデスクトップアプリで、クローンしたフォルダを開いて「燃料帳をセットアップして」と頼みます。シートのURLを聞かれるので①のURLを渡します。年齢・身長などが空なら聞かれます。最後に自分専用のページのリンクが出ます。

### ④ iPhoneのSafariで開いてホーム画面に追加する

1. ③で出たリンクを**iPhoneのSafari**で開き、claude.aiにサインインします
2. 共有ボタン → **「ホーム画面に追加」**
3. 初回は、Google Sheets、Strava、Claudeの利用を許可する画面が出るので許可します

使い方：画面下の **「＋ 記録」** → 写真ボタンで撮影かライブラリから選ぶ → メモに量や中身を書く（例：ご飯130g、揚げ物は鶏ささみ）→ **推定する** → 数字を確かめて **スプレッドシートに記録**。

### スキルを使わない手動の手順

1. ①②を済ませます
2. `app/fuel-ledger.html` をコピーし、先頭近くの `const SPREADSHEET_ID = "__SPREADSHEET_ID__";` を自分のシートのID（URLの `/d/` と次の `/` の間）に置き換えます
3. 「設定」タブに年齢・性別・身長・体重・体脂肪率を書きます
4. Claude Codeかデスクトップアプリで、「このHTMLをアーティファクトとして公開して。機能は次のとおり」と頼み、次の宣言を渡します

   ```json
   {
     "mcp": {"servers": [
       {"server": "Google Sheets", "tools": ["get_values", "append_values"]},
       {"server": "Strava", "tools": ["list_activities"]}
     ]},
     "sample": {"images": true}
   }
   ```

5. ④のとおりiPhoneで開きます

### 更新するとき

リポジトリを `git pull` してから、「燃料帳を更新して」と頼みます。前に公開したアーティファクトのURLを渡して公開し直すので、**リンクは変わりません**。

## Withingsの体重を入れる

任意です。燃料帳のシートに「Withings」タブがあれば、体重の推移と週ごとの体脂肪量を表示し、基本消費の計算に直近7日の平均体重・体脂肪率を使います。

標準の手順はIFTTTのアプレット（Withings → Google Sheets）で別のファイルに書き込み、`IMPORTRANGE` で「Withings」タブに読み込む方法です。詳しくは [docs/withings-ifttt.md](docs/withings-ifttt.md)。Apps ScriptでWithingsのAPIから書き込んでいる形式（A＝日時、B＝体重、C＝体脂肪率）もそのまま読めます。

## 計算のルール（要約）

- **基本消費**＝基礎代謝（Mifflin-St Jeor式）×活動係数（出社日1.5、在宅日1.4）を100 kcal単位に丸めたもの。「設定」タブで上書きできます
- **運動の上乗せ**：ラン＝Strava×0.93、スイム＝泳いだ時間×8.5 kcal/分、Zwift＝×0.8、自転車＝×0.6、筋トレ＝×0.7、その他＝×0.7
- **摂取目標**＝基本消費＋上乗せ−赤字目標（既定300 kcal）
- **エネルギー利用可能量の下限**＝除脂肪量×30 kcal
- **たんぱく質**＝体重×1.8〜2.0 g、**脂質**＝70 g以下、**炭水化物**＝体重×4 g（上乗せ400 kcal以上の日は×5 g）
- ナッツは吸収率を考慮して約15%（アーモンドは20%）差し引いて推定

根拠と詳細は [docs/calculation-rules.md](docs/calculation-rules.md)。

## 制限

- **iPhoneのClaudeアプリでは、連携（Google Sheets、Strava）が動かない可能性が高い**です。Safariでclaude.aiのリンクを開いて使ってください
- **連携つきのアーティファクトは、他の人と共有できません。** 誰かが公開したリンクをもらっても使えないので、各自がセットアップしてください
- 出社・在宅の切り替えは、**端末（ブラウザ）ごとに記憶**されます。iPhoneとパソコンで別々です
- 記録した行の**修正・削除はスプレッドシートで**行ってください。ページは追加だけです
- 個人設定の変更もスプレッドシートの「設定」タブで行い、ページの ↻ で読み込み直します

## 注意

- 燃料帳は**医療や栄養指導の代わりではありません。** 持病がある人、妊娠中の人、食事制限を指示されている人は、医師や管理栄養士に相談してください
- 写真からの推定値には**誤差があります**（特に外食や量の分からない料理）。確度「低」の記録は参考程度に見てください
- **Stravaのデータはページの中の計算にだけ使い、Claudeには送りません**（Stravaの規約による）。Claudeに送るのは、食事の写真・メモ・「設定」タブの年齢などの属性・よく食べる市販品の表です

## ライセンス

[MIT](LICENSE)
