---
routerMode: hash
theme: seriph
title: AI時代にTemporalから学ぶこと
info: |
  ## AI時代にTemporalから学ぶこと
  社内勉強会 2026/06/03

class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
---

# AI時代にTemporalから学ぶこと

APIの「使い方」ではなく「設計思想」を学ぶ

中村崇人

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/aTakatoNakamura" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---

# 目次

1. AIがコードを書く時代に何を学ぶ？
2. 題材：JavaScript Temporal API
3. Dateの何が問題だったか
4. なぜ30年間放置されたのか
5. Temporalはどう解決したか
6. **「時間」という概念の複雑さ**
7. AI時代に「人間」が考えること
8. まとめ

---

## layout: section

# AIがコードを書く時代に<br>何を学ぶ？

---

# こんな経験ありませんか？

<v-clicks>

- APIのメソッド名を覚えたのに、すぐ忘れる
- ドキュメントを見ればわかることを暗記しようとしていた
- Copilotに聞けば一瞬で出てくるコードを手で書いていた

</v-clicks>

<v-click>

### AI時代、これらの「暗記」の価値は下がっている

</v-click>

---

# では何を学ぶべきか？

<div class="grid grid-cols-2 gap-8 mt-8">
<div>

### 価値が下がるもの

- APIのメソッド名
- 引数の順番
- 細かい文法
- ボイラープレート

</div>
<div>

### 価値が上がるもの

- **なぜそう設計されたか**
- **どんな問題を解決するか**
- **トレードオフの判断**
- **設計の良し悪しを見抜く力**

</div>
</div>

<v-click>

<div class="mt-8 text-center text-xl">

**「使い方」ではなく「設計の背景」を学ぶ**

</div>

</v-click>

---

# 今日の題材：Temporal API

JavaScriptの`Date`を置き換える新しい日付・時刻API

<v-clicks>

- 2024年にTC39 Stage 3、2025年にStage 4（標準化完了）
- 2026年現在、主要ブラウザでネイティブサポート
- **30年間の問題を解決**するために設計された

</v-clicks>

<v-click>

> "Date has been a long-standing pain point in ECMAScript"
> — TC39 Temporal Proposal

</v-click>

---

## layout: section

# Dateの何が問題だったか

問題を正しく言語化する

---

# Dateの歴史的背景

1995年、Brendan Eichは**10日間**でJavaScriptを設計した

<v-click>

> "Brendan, under orders to 'make it like Java' copied the date object from the existing, infant, java.util.Date implementation. **This implementation was frankly terrible.**"
> — Maggie Pint (Temporal Champion)

</v-click>

<v-click>

- そのJavaのDateは**1997年に非推奨化**
- JavaScriptは**30年間そのまま**

</v-click>

---

# TC39が特定した問題リスト

Temporalチャンピオンたちが最初に整理した問題：

<v-clicks>

1. ユーザーのローカル時間とUTC以外のタイムゾーンをサポートしない
2. パーサーの挙動が**信頼できないほど不安定**
3. **Dateオブジェクトがミュータブル**
4. DSTの挙動が予測不能
5. 計算APIが扱いにくい
6. 非グレゴリオ暦をサポートしない

</v-clicks>

<v-click>

> 出典: [Fixing JavaScript Date](https://maggiepint.com/2017/04/09/fixing-javascript-date-getting-started/)

</v-click>

---

# 問題1: ミュータブル

```javascript
const meeting = new Date("2026-06-03T10:00:00");

// 別の関数に渡す
scheduleReminder(meeting);

// あれ？meetingが変わってる！
console.log(meeting); // 予期しない値に...
```

<v-click>

### 問題の本質

- オブジェクトを共有すると、どこかで変更される可能性
- 「この日付は変わらない」という保証がない
- デバッグが困難

</v-click>

---

# 問題2: 一つのクラスで全部やろうとする

`Date`は以下をすべて1つのクラスで扱う

- 絶対時刻（タイムスタンプ）
- 日付と時刻の組み合わせ
- タイムゾーン変換

<v-click>

### 問題の本質

```javascript
// 誕生日を保存したい（時刻は不要）
const birthday = new Date("1990-05-15");

// でもDateは常に時刻を持つ
console.log(birthday.toISOString());
// 1990-05-15T00:00:00.000Z ← 勝手に00:00:00が付く
```

→ **概念の区別がない**ので、意図しない変換が起きる

</v-click>

---

# 問題3: 暗黙の変換・曖昧さ

```javascript
// 月が0始まり（Javaからの設計ミスをコピー）
new Date(2026, 6, 3); // 7月3日になる

// 文字列パースが不安定
new Date("2026-06-03"); // UTCとして解釈
new Date("2026/06/03"); // ローカル時間として解釈

// 無効な日付もエラーにならない
new Date("not a date"); // Invalid Date（実行時まで気づかない）
```

<v-click>

### 問題の本質

- **暗黙の変換**が多すぎる
- **失敗が静か**（エラーにならない）
- 開発者が常に「罠」を意識しないといけない

</v-click>

---

# 問題の整理

| 問題         | 本質                         |
| ------------ | ---------------------------- |
| ミュータブル | 共有時の予測不能性           |
| 一つで全部   | 概念の区別がない             |
| 暗黙の変換   | 意図しない動作が静かに起きる |

<v-click>

### これらは「Date固有の問題」ではない

**API設計における普遍的なアンチパターン**

</v-click>

---

## layout: section

# なぜ30年間放置されたのか

Web互換性の呪縛

---

# 「壊せない」という制約

問題がわかっているのに、なぜ直せなかったのか？

<v-clicks>

- 世界中のWebサイトが`Date`に依存している
- `setMonth()`の挙動を変えると、**既存のコードが壊れる**
- 月を0始まりから1始まりに変える？→ **世界中のコードがバグる**
- ブラウザベンダーは「Webを壊す」変更を拒否する

</v-clicks>

<v-click>

> これが**Web互換性 (Web Compatibility)** の問題

</v-click>

---

# Temporalの戦略：「置き換える」のではなく「並立」

Dateを**修正しない**という選択

<v-clicks>

- `Date`はそのまま残す（deprecatedにもしない）
- 完全に新しい名前空間`Temporal`を作る
- 新しいコードは徐々に移行すればいい
- 古いコードは動き続ける

</v-clicks>

<v-click>

### 設計上の教訓

**破壊的変更より、新しい抽象を追加する**

→ これはライブラリ設計、API設計全般に通じる考え方

</v-click>

---

# 10年かかった標準化

Temporalのタイムライン：

| 年        | 出来事                |
| --------- | --------------------- |
| 2017      | 最初の提案・議論開始  |
| 2020      | Stage 2（ドラフト）   |
| 2021-2023 | 大幅なAPI見直し       |
| 2024      | Stage 3（実装段階）   |
| 2025      | Stage 4（標準化完了） |

<v-click>

### なぜこんなに時間がかかった？

- 日付・時刻は**複雑な問題領域**（タイムゾーン、DST、暦...）
- 「正しく」設計するには**十分な議論**が必要だった
- 急いで作ると、またDateのようになる

</v-click>

---

## layout: section

# Temporalはどう解決したか

設計判断から学ぶ

---

# TC39 Proposalの設計原則

Temporalは以下の原則で設計された：

<v-clicks>

- **All Temporal objects are immutable.**（すべてイミュータブル）
- ローカルの暦システムをサポートするが、グレゴリオ暦との変換が可能であること
- すべての時刻は24時間制に基づく
- うるう秒は表現しない

</v-clicks>

<v-click>

> 出典: [TC39 Temporal Proposal - Principles](https://github.com/tc39/proposal-temporal#principles)

</v-click>

---

# 解決策1: イミュータブル

```javascript
const meeting = Temporal.PlainDateTime.from("2026-06-03T10:00");

// 変更は常に新しいオブジェクトを返す
const rescheduled = meeting.add({ hours: 2 });

console.log(meeting.toString()); // 2026-06-03T10:00:00（元のまま）
console.log(rescheduled.toString()); // 2026-06-03T12:00:00（新しい値）
```

<v-click>

### 設計判断

- **変更を禁止**することで予測可能性を確保
- パフォーマンスより**安全性を優先**
- 関数型プログラミングの考え方を採用

</v-click>

---

# 解決策2: 用途別のクラス

```
Temporal.Instant      → 絶対時刻（タイムスタンプ）
Temporal.ZonedDateTime → タイムゾーン付き日時
Temporal.PlainDateTime → 日付と時刻（タイムゾーンなし）
Temporal.PlainDate    → 日付だけ
Temporal.PlainTime    → 時刻だけ
Temporal.Duration     → 期間
```

<v-click>

### 設計判断

- **概念ごとに型を分ける**ことで意図を明確に
- 「日付だけ」なら`PlainDate`、「時刻も必要」なら`PlainDateTime`
- **型が制約になる** → 間違った使い方がしにくい

</v-click>

---

# 解決策3: 明示的・厳格

```javascript
// 月は1始まり（直感的）
Temporal.PlainDate.from({ year: 2026, month: 6, day: 3 });

// 無効な値は即座に例外
Temporal.PlainDate.from("not a date"); // RangeError!

// タイムゾーンは常に明示
Temporal.ZonedDateTime.from({
  year: 2026,
  month: 6,
  day: 3,
  hour: 10,
  timeZone: "Asia/Tokyo", // 必須
});
```

<v-click>

### 設計判断

- **暗黙より明示**を優先
- **早く失敗**する（Fail Fast）
- 開発者が「罠」を意識しなくていい

</v-click>

---

# 設計判断の整理

| Dateの問題   | Temporalの解決策 | 設計原則         |
| ------------ | ---------------- | ---------------- |
| ミュータブル | イミュータブル   | 予測可能性の確保 |
| 一つで全部   | 用途別のクラス   | 概念の分離       |
| 暗黙の変換   | 明示的・厳格     | Fail Fast        |

<v-click>

### これらは「Temporal固有の解決策」ではない

**良いAPI設計における普遍的なパターン**

</v-click>

---

## layout: section

# 「時間」という概念の複雑さ

なぜTemporalはこんなに型が多いのか

---

# 「時間」は技術の問題ではない

そもそも日付・時刻は**人間が作り上げた複雑な概念**

<v-clicks>

- **「瞬間」と「日付」は別の概念**
  - 「2026年6月3日」は世界中で同時に来ない
  - でも「今この瞬間」は世界共通
- **タイムゾーンは政治的に決まる**
  - 国境の変更、法律の改正で変わる
  - 過去のタイムゾーンと今のタイムゾーンは違う
- **カレンダーは文化によって違う**
  - グレゴリオ暦、和暦、イスラム暦、ヘブライ暦...
- **夏時間（DST）は人為的なルール**
  - 年によって変わる。廃止する国も。

</v-clicks>

---

# Dateが失敗した理由

「シンプルにしすぎた」

<v-click>

```javascript
// Dateは「一つの数値（ミリ秒）」で全部を表現しようとした
const d = new Date();
d.getTime(); // → 1748941234567（瞬間）
d.getFullYear(); // → 2026（ローカルの年）
d.getMonth(); // → 5（ローカルの月）
```

</v-click>

<v-click>

### 問題の本質

- 「瞬間」と「日付」と「時刻」を**区別しなかった**
- タイムゾーンを**暗黙的に**処理した
- 複雑な現実を**シンプルなモデル**に押し込めた

→ その結果、**開発者が複雑さを管理する羽目に**

</v-click>

---

# Temporalは「現実の複雑さ」を正直に反映した

```
Temporal.Instant      → 「瞬間」だけを表す
Temporal.PlainDate    → 「日付」だけを表す（タイムゾーンなし）
Temporal.PlainTime    → 「時刻」だけを表す
Temporal.ZonedDateTime → 「タイムゾーン付きの日時」を表す
```

<v-click>

### 型が多い理由

- 現実世界で**区別される概念**を、コードでも**区別する**
- 「日付だけ」と「瞬間」を混同できない設計
- 複雑さを**隠す**のではなく**表現する**

</v-click>

<v-click>

> **シンプルなAPIが正解とは限らない**
> 問題領域が複雑なら、APIも複雑になって良い

</v-click>

---

# 「2026年6月3日10時」の解釈

同じ文字列でも、意味が違う可能性がある

<v-click>

| 質問                         | 使うべき型             |
| ---------------------------- | ---------------------- |
| 「その瞬間」を記録したい     | `ZonedDateTime`        |
| 「日本時間の6/3 10時」に会議 | `ZonedDateTime`        |
| 「誕生日」を記録したい       | `PlainDate`            |
| 「毎朝10時」のアラーム       | `PlainTime`            |
| 「2時間後」を計算したい      | `Duration` + `Instant` |

</v-click>

<v-click>

### これを「Date」で全部やろうとするから問題が起きた

Temporalは**「何を表現したいか」を開発者に考えさせる**

</v-click>

---

# 技術より「モデリング」が難しい

<v-clicks>

- Temporalの型設計は、**日付・時刻の概念を正確にモデリング**した結果
- 「コードを書く」前に「概念を理解する」必要がある
- これは**プログラミング言語の知識**では解決できない

</v-clicks>

<v-click>

### AIはこの「概念の境界」を理解しているか？

- `PlainDate`と`ZonedDateTime`の違いを説明できる？
- ユーザーの意図から適切な型を選べる？
- タイムゾーンの罠を予測できる？

→ **人間が概念を理解し、AIに正しく伝える必要がある**

</v-click>

---

## layout: section

# AI時代に「人間」が考えること

---

# AIは「なぜ」を理解しているわけではない

AIができることと、人間がやるべきこと

<div class="grid grid-cols-2 gap-8 mt-4">
<div>

### AIが得意なこと

- パターンの学習・再現
- 既存のAPIの使い方
- ボイラープレートの生成
- ドキュメントの要約

</div>
<div>

### 人間がやるべきこと

- **問題の構造**を見抜く
- **トレードオフ**を判断する
- **設計の良し悪し**を見極める
- **「なぜ」を説明**できる

</div>
</div>

<v-click>

### AIは「Dateがミュータブルなのはなぜ問題か」を本当に理解している？

</v-click>

---

# 設計思想を学ぶ = AIを「使う側」の能力を高める

<v-clicks>

- **問題を言語化**できれば、AIに適切な指示ができる
- **トレードオフを理解**していれば、AIの出力を評価できる
- **設計原則を知って**いれば、AI生成コードのレビューができる

</v-clicks>

<v-click>

### 具体例

```
AIへの指示:
「日付を操作する関数を作って」
→ 曖昧。Dateかもしれないし、破壊的かもしれない

「Temporalでイミュータブルに日付を操作する関数を作って」
→ 明確。意図が伝わる
```

</v-click>

---

# 「AIに任せる」と「AIを使う」の違い

<div class="grid grid-cols-2 gap-8 mt-4">
<div>

### AIに任せる

- 「いい感じにやって」
- 結果をそのまま使う
- なぜそのコードかわからない
- 問題が起きたらまたAIに聞く

</div>
<div>

### AIを使う

- 問題を明確にして指示
- 結果を評価できる
- なぜその設計か説明できる
- **人間が責任を持つ**

</div>
</div>

<v-click>

### 設計思想を学ぶことは、AIを「使う側」になるための投資

</v-click>

---

## layout: section

# まとめ

---

# AI時代に学ぶべきこと

### APIの「使い方」より「設計の背景」

<v-clicks>

- **問題を正しく言語化する力**
  - Dateの何が問題だったか？
- **設計判断を理解する力**
  - Temporalはなぜそう設計されたか？
- **制約とトレードオフを理解する力**
  - なぜ30年かかったのか？
- **原則を他に応用する力**
  - ライブラリ選定、AI指示、自分の設計

</v-clicks>

---

# Temporalから学んだこと

| 学び               | 説明                                      |
| ------------------ | ----------------------------------------- |
| 問題領域の複雑さ   | 時間は技術ではなく「人間の概念」の問題    |
| モデリングの重要性 | 現実世界の区別をコードでも区別する        |
| シンプル≠正解      | 複雑な問題には複雑なAPIが必要なこともある |

<v-click>

### 「イミュータブル」「概念の分離」は手段であり、目的ではない

本質は**「現実世界を正しくモデリングする」**こと

</v-click>

---

# 最後に

<v-click>

> AIがコードを書く時代、
> 私たちは**「何を書くか」より「何を表現したいか」**を
> 考える側になる

</v-click>

<v-click>

Temporalはその学びに最適な題材だった

- 問題領域が複雑（時間という概念）
- 解決策が誠実（複雑さを隠さない）
- 人間の理解が必要（AIには難しい判断）

</v-click>

---

## layout: center

# 参考リンク

- [TC39 Temporal Proposal](https://tc39.es/proposal-temporal/docs/)
- [Fixing JavaScript Date - Getting Started](https://maggiepint.com/2017/04/09/fixing-javascript-date-getting-started/) - Maggie Pint
- [MDN Temporal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal)

---

## layout: end

# ありがとうございました
