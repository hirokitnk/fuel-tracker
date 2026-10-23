---
name: fuel-ledger-setup
description: 燃料帳（fuel-ledger）を自分のclaude.aiアカウントにアーティファクトとして公開するときに使う。「燃料帳をセットアップして」「燃料帳を更新して」と頼まれたら、コネクタの確認、スプレッドシートの確認と設定、ページの公開までを行う。
---

# 燃料帳のセットアップ

燃料帳は、食事の写真とメモからClaudeが栄養を推定してGoogleスプレッドシートに記録し、Stravaの運動とWithingsの体重から日々のエネルギー収支を表示するページです。連携つきのアーティファクトは他の人と共有できないので、**使う人が自分のアカウントで公開する**必要があります。このスキルはその手順です。

ページ本体は、このスキルと同じリポジトリの `app/fuel-ledger.html` です。手元にない場合は https://github.com/hirokitnk/fuel-ledger から取得してください（`git clone` か、`https://raw.githubusercontent.com/hirokitnk/fuel-ledger/main/app/fuel-ledger.html` を保存）。

## 1. 前提を確かめる

1. このセッションのツール一覧に、Google SheetsとStravaのコネクタのツールがあるか確かめる。名前は `mcp__<id>__get_values`、`mcp__<id>__append_values`、`mcp__<id>__list_activities` のような形で出る。どのidがどのコネクタかは、`artifact-capabilities` スキルを読むと分かる。
   - ない場合は、「claude.aiの設定 → コネクタで、Google SheetsとStravaを追加してから、このセッションを開き直してください」と伝えて止まる。StravaのコネクタにはStravaの有料プランが必要。
2. Artifactツールが使えるか確かめる（Claude CodeかClaudeデスクトップアプリ）。使えなければ、そう伝えて止まる。
3. `artifact-capabilities` スキルを読み、`mcp` と `sample` が使える機能に入っていることを確かめる。

## 2. スプレッドシートを確かめる

1. 先にREADMEの「テンプレートをコピー」のリンクからシートをコピーしてもらい、そのURLを聞く。
2. URLの `/d/` と次の `/` の間をスプレッドシートIDとして取り出す。
3. `get_values` で `食事ログ!A1:H2` と `設定!A1:J200` を読む。
   - 読めないタブがあれば、どのタブがないかを伝え、作ってよいか聞く。了承を得たら、Google Sheetsのコネクタのタブ追加のツール（`update_spreadsheet` など。ツールの説明を読んで選ぶ）で作り、`template/README.md` の見出しと既定値を `update_values` で書く。
   - `食事ログ` の見出しは `日付, 区分, 内容, エネルギー(kcal), たんぱく質(g), 脂質(g), 炭水化物(g), 確度`。
4. `Withings!A1:E5` も読んでみる。なければ任意であることと、`docs/withings-ifttt.md` の手順を伝える。

## 3. 設定を埋める

「設定」タブの `age`、`sex`、`height_cm`、`weight_kg`、`body_fat_pct` が空なら、本人に値を聞く。書き込む内容（セル番地と値）を見せて**同意を得てから** `update_values` で書く。体重と体脂肪率は、Withingsタブがあればそちらの直近7日平均が優先されることも伝える。`goal_note`（目標の大会など）も任意で聞く。

基本消費の自動計算（Mifflin-St Jeor式×活動係数）の結果を伝え、実感と違えば `base_office_kcal`／`base_home_kcal` で上書きできると伝える。

## 4. ページを公開する

1. `app/fuel-ledger.html` を読み、`const SPREADSHEET_ID = "__SPREADSHEET_ID__";` の `__SPREADSHEET_ID__` を手順2のIDに置き換えたコピーを、スクラッチパッドなど作業用のディレクトリに書く（リポジトリの元ファイルは変えない）。ファイル名は `fuel-ledger.html` のままにする。
2. ページが呼ぶツール名（`get_values`、`append_values`、`list_activities`）が、このセッションのコネクタのツール名と一致するか確かめる。違う名前なら、ページの `callTool(SHEETS, "...")`／`callTool(STRAVA, "...")` と下の宣言を合わせて直し、何を変えたか本人に伝える。引数の名前（`spreadsheetId`、`range`、`values`、`first`、`range_start`、`range_end`）もツールの入力スキーマと照らし合わせる。
3. Artifactツールで公開する。初回は `icon: "food"` をつけ、次の機能を宣言する。

   ```json
   {
     "mcp": {"servers": [
       {"server": "Google Sheets", "tools": ["get_values", "append_values"]},
       {"server": "Strava", "tools": ["list_activities"]}
     ]},
     "sample": {"images": true}
   }
   ```

   `server` はコネクタの表示名そのもの（`artifact-capabilities` スキルが示す名前）にする。
4. 公開できたらリンクを伝える。ページを開くと、Google Sheets、Strava、Claudeの利用を許可する画面が順に出ることを伝える。

## 5. iPhoneでの使い方を伝える

- 公開したリンクを**iPhoneのSafari**で開き、claude.aiにサインインする。iPhoneのClaudeアプリでは連携が動かない可能性が高い。
- 共有ボタン → 「ホーム画面に追加」で、アプリのように開ける。
- 初回は、Google Sheets、Strava、Claudeの利用を許可する画面が出るので許可する。
- 画面下の「＋ 記録」から写真を撮るかライブラリから選び、「推定する」→ 数字を確かめて「スプレッドシートに記録」。

## 6. 更新するとき

リポジトリの `app/fuel-ledger.html` が新しくなったら、同じ手順4で、**前に公開したアーティファクトのURLを `url` に渡して**公開し直す。URLが分からなければ Artifactツールの `action: "list"` で探す。こうするとリンクは変わらず、ホーム画面のアイコンもそのまま使える。機能の宣言を変えないなら `capabilities` は省いてよい（前の宣言が引き継がれる）。
