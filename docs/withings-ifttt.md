# IFTTTでWithingsの体重をシートに入れる

Withingsの体重計で測るたびに、IFTTTが体重をGoogleスプレッドシートに1行ずつ書き込むようにします。燃料帳はそれを読んで、体重の推移と基本消費の計算に使います。

## 先に知っておくこと

- **IFTTTが書き込むのは、燃料帳のシートとは別のファイルです。** IFTTTの「Add row to spreadsheet」は、Googleドライブ上のファイル名とフォルダを指定して、そのファイルの最初のシートに行を足します。既存のファイルの特定のタブには書けません。そこで、燃料帳のシートの「Withings」タブから `IMPORTRANGE` で読み込みます（下の手順4）。
- IFTTTの無料プランでは、確認が1時間おきです。測ってからページに出るまで最大1時間ほどかかります。
- IFTTTは2,000行ごとに新しいファイルを作ります。毎日1回測るなら5年以上もちますが、新しいファイルができたら手順4の参照先を変えてください。
- 実際の日付の書式と単位は、**最初の1行が書き込まれたら必ず目で確かめてください。** この手順は、IFTTTのWithingsトリガーが出す項目（`DateAndTime`、`Weight`、`Unit`、`FatMass`、`FatMassPercent`、`LeanMass`）にもとづいていますが、作者は出力をすべての環境で確かめたわけではありません。

## 手順

1. IFTTTで新しいアプレットを作ります。If This＝**Withings → New weight measurement**、Then That＝**Google Sheets → Add row to spreadsheet**。
2. 「Formatted row」を次のようにします（列の並びがA＝日時、B＝体重、C＝体脂肪率、D＝単位になります）。

   ```
   {{DateAndTime}} ||| {{Weight}} ||| {{FatMassPercent}} ||| {{Unit}}
   ```

   ファイル名は `withings-log` など分かりやすいものに、フォルダは `IFTTT/Withings` など既定のままで構いません。
3. 一度体重計に乗って、ドライブにファイルができ、1行目に `January 15, 2026 at 07:30AM | 65.2 | 18.5 | kg` のような行が入ることを確かめます。
4. 燃料帳のシートに「Withings」タブを作り、A1に次の式を入れます（`<IFTTTのファイルのURL>` は手順3のファイルのURL）。初回は「アクセスを許可」を押します。

   ```
   =IMPORTRANGE("<IFTTTのファイルのURL>", "A1:D")
   ```

5. 燃料帳のページで ↻ を押すと、体重の欄が出ます。

## うまく読めないとき

- **列の並びを変えた**：「設定」タブに `withings_col_date`、`withings_col_weight`、`withings_col_fat` を追加して、列の文字（A、Bなど）を書きます。体脂肪率がなければ `withings_col_fat` を空欄に。
- **単位がlbになっている**：行に `lbs` と書かれていれば自動でkgに換算します。単位の列を入れていない場合は、Withingsのアプリで単位をkgにしてください。
- **日付が読めない**：読めるのは `2026-01-15 …`、`2026/01/15 …`、`January 15, 2026 at …` の形です。ほかの形なら、IMPORTRANGEの代わりに `=ARRAYFORMULA(...)` などで `YYYY-MM-DD` に直した列を作り、`withings_col_date` でそちらを指定してください。

## すでにApps ScriptでWithingsのAPIから書き込んでいる場合

A＝日時（`2026-01-15 07:30`）、B＝体重(kg)、C＝体脂肪率(%) の形なら、そのまま「Withings」タブとして読めます。
