# タクシー乗務タイマー 実装プラン

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** タクシー乗務における11分以上の休憩記録・連続走行6時間制限のリアルタイム可視化Webアプリを、単一HTMLファイルで実装しGitHub Pagesに公開する。

**Architecture:** 単一 `index.html` ファイル内に HTML + CSS + Vanilla JS を内包。localStorage で記録を永続化。リアルタイム更新は `setInterval` による1秒ティック。依存ライブラリ・ビルドツールなし。

**Tech Stack:** HTML5, CSS3, Vanilla JavaScript (ES2020+), localStorage, GitHub Pages

---

## ファイル構成

```
taxi-timer/
├── index.html              # アプリ本体（HTML+CSS+JS全部入り）
├── icon-180.png            # apple-touch-icon（暫定で単色画像）
├── README.md               # 概要とデプロイ手順
├── .gitignore              # macOS用 .DS_Store 除外
└── docs/
    ├── superpowers/
    │   ├── specs/2026-04-24-taxi-timer-design.md  (既存)
    │   └── plans/2026-04-24-taxi-timer.md          (this file)
```

### 単一ファイル内の論理セクション

`index.html` 内のJSは以下のセクションに分けてコメント区切りで整理する。

| セクション | 責務 |
|---|---|
| `// --- Storage ---` | localStorage の読み書き |
| `// --- Pure ---` | 純粋関数（時刻計算・フォーマット） |
| `// --- State ---` | アプリ状態（ストップウォッチ・記録配列） |
| `// --- Render ---` | DOM更新 |
| `// --- Events ---` | ボタンハンドラ |
| `// --- Tick ---` | setInterval による1秒更新 |
| `// --- Init ---` | 起動時処理 |

---

## 検証方針

テストフレームワークは導入しない（spec §4.1 「依存なし」方針）。代わりに：

- **純粋関数**: `index.html` の `<script>` 末尾に `// --- DevTest ---` セクションを暫定的に置き、`console.assert` で検証する。確認後そのセクションは削除してコミットする
- **UI/挙動**: 各タスクで明示的なブラウザ手動検証ステップを含める
- **コミット**: 各タスク末尾でコミット

---

## Task 1: リポジトリ初期化と基本ファイル

**Files:**
- Create: `README.md`
- Create: `.gitignore`
- Create: `index.html`（骨組みのみ）

- [ ] **Step 1: `.gitignore` を作成**

```
.DS_Store
*.swp
.vscode/
```

- [ ] **Step 2: `README.md` を作成**

```markdown
# タクシー乗務タイマー

タクシー乗務中の休憩管理Webアプリ。11分以上の停車を休憩として記録し、前回記録から6時間以内に次の休憩を取るようカウントダウンする。

## 使用

公開URL: https://hidenaka.github.io/taxi-timer/

iPad Safariで開き、共有 → ホーム画面に追加でアイコン化。

## 開発

単一HTMLファイル。`index.html` をブラウザで直接開けば動作する。依存・ビルドなし。
```

- [ ] **Step 3: `index.html` 骨組みを作成**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
  <title>タクシー乗務タイマー</title>
</head>
<body>
  <h1>タクシー乗務タイマー</h1>
</body>
</html>
```

- [ ] **Step 4: ブラウザで検証**

`index.html` をFinder上でダブルクリック → Safariで開く。タイトルとh1が表示されること。

- [ ] **Step 5: コミット**

```bash
git add README.md .gitignore index.html
git commit -m "chore: initialize project skeleton"
```

---

## Task 2: HTML静的レイアウト構築

**Files:**
- Modify: `index.html`（`<body>` を差し替え）

- [ ] **Step 1: 静的HTMLを実装**

`index.html` の `<body>` を以下で置き換え：

```html
<body>
  <main>
    <header>
      <h1>タクシー乗務タイマー</h1>
      <p id="today-date">----/--/-- (-)</p>
    </header>

    <section class="shift-start">
      <label for="shift-start-input">乗務開始:</label>
      <input type="time" id="shift-start-input" value="07:00">
    </section>

    <section class="stopwatch">
      <div id="stopwatch-display">00:00:00</div>
      <div class="buttons">
        <button id="btn-start" type="button">スタート</button>
        <button id="btn-record" type="button" disabled>記録</button>
        <button id="btn-discard" type="button" disabled>破棄</button>
      </div>
    </section>

    <section class="metrics">
      <p>連続走行可能時刻: <span id="deadline-time">--:--</span></p>
      <p>連続走行可能時間: <span id="deadline-remaining">--</span></p>
      <p>残り必要休憩: <span id="break-remaining">180分 (3時間0分)</span></p>
      <p>合計休憩: <span id="break-total">0分 (0時間0分)</span></p>
    </section>

    <section class="history">
      <h2>記録履歴 (<span id="history-count">0</span>件)</h2>
      <ul id="history-list"></ul>
    </section>

    <section class="reset">
      <button id="btn-reset" type="button">全リセット</button>
    </section>
  </main>
</body>
```

- [ ] **Step 2: ブラウザで検証**

リロードして全セクションが縦に並んで表示されること。記録・破棄ボタンはグレーアウトしていること。

- [ ] **Step 3: コミット**

```bash
git add index.html
git commit -m "feat: add static HTML layout"
```

---

## Task 3: CSS スタイリング

**Files:**
- Modify: `index.html`（`<head>` 内に `<style>` 追加）

- [ ] **Step 1: CSS を `<head>` 内に追加（`</head>` の直前）**

```html
<style>
  * { box-sizing: border-box; }
  html, body { margin: 0; padding: 0; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", sans-serif;
    background: #1a1a1a;
    color: #f0f0f0;
    line-height: 1.5;
  }
  main {
    max-width: 600px;
    margin: 0 auto;
    padding: 16px;
  }
  header h1 { font-size: 20px; margin: 0 0 4px; }
  header p { font-size: 14px; margin: 0 0 16px; color: #aaa; }

  section { margin-bottom: 20px; padding: 16px; background: #2a2a2a; border-radius: 12px; }

  .shift-start label { font-size: 16px; margin-right: 8px; }
  .shift-start input[type="time"] {
    font-size: 18px;
    padding: 8px 12px;
    background: #1a1a1a;
    color: #f0f0f0;
    border: 1px solid #444;
    border-radius: 8px;
  }

  .stopwatch { text-align: center; }
  #stopwatch-display {
    font-size: 56px;
    font-variant-numeric: tabular-nums;
    font-weight: 300;
    margin: 8px 0 16px;
    letter-spacing: 2px;
  }
  .buttons { display: flex; gap: 8px; justify-content: center; flex-wrap: wrap; }
  button {
    min-height: 48px;
    min-width: 96px;
    padding: 0 20px;
    font-size: 16px;
    border: none;
    border-radius: 8px;
    background: #3a7bd5;
    color: white;
    cursor: pointer;
  }
  button:disabled { background: #555; color: #888; cursor: not-allowed; }
  #btn-record { background: #2ea043; }
  #btn-discard { background: #8b2e2e; }
  #btn-reset { background: #555; width: 100%; }

  .metrics p { margin: 8px 0; font-size: 16px; }
  .metrics span { font-weight: 600; font-variant-numeric: tabular-nums; }

  .history h2 { font-size: 16px; margin: 0 0 12px; }
  #history-list { list-style: none; padding: 0; margin: 0; }
  #history-list li {
    padding: 8px 0;
    border-bottom: 1px solid #3a3a3a;
    font-variant-numeric: tabular-nums;
  }
  #history-list li:last-child { border-bottom: none; }

  .warn { color: #ff9500; }
  .danger { color: #ff3b30; }
  .over { background: #5a0e0e; border: 2px solid #ff3b30; }
</style>
```

- [ ] **Step 2: ブラウザで検証**

リロード。ダークテーマの整った縦レイアウトで表示されること。ストップウォッチは大きく表示されること。

- [ ] **Step 3: iPad miniサイズでも見え方を確認**

ChromeまたはSafariの開発者ツールでデバイスをiPad mini（768×1024）に切替えて確認。

- [ ] **Step 4: コミット**

```bash
git add index.html
git commit -m "feat: add dark theme CSS styling"
```

---

## Task 4: 純粋関数（時刻計算・フォーマット）

**Files:**
- Modify: `index.html`（`</body>` の直前に `<script>` 追加）

- [ ] **Step 1: 純粋関数を実装**

`</body>` の直前に以下を追加：

```html
<script>
// ============================================================
// --- Pure ---
// 依存のない純粋関数。入出力のみで動く。
// ============================================================

// 秒 → "HH:MM:SS"
function fmtStopwatch(totalSec) {
  const s = Math.max(0, Math.floor(totalSec));
  const h = Math.floor(s / 3600);
  const m = Math.floor((s % 3600) / 60);
  const sec = s % 60;
  return `${String(h).padStart(2, '0')}:${String(m).padStart(2, '0')}:${String(sec).padStart(2, '0')}`;
}

// 分 → "X分 (Y時間Z分)"
function fmtMinutes(minutes) {
  const m = Math.max(0, Math.floor(minutes));
  const h = Math.floor(m / 60);
  const rem = m % 60;
  return `${m}分 (${h}時間${rem}分)`;
}

// ミリ秒 → "X時間Y分" （残り時間用）負なら "超過: X時間Y分"
function fmtRemaining(ms) {
  const abs = Math.abs(ms);
  const totalMin = Math.floor(abs / 60000);
  const h = Math.floor(totalMin / 60);
  const m = totalMin % 60;
  const core = `${h}時間${m}分`;
  return ms < 0 ? `超過: ${core}` : core;
}

// Date → "HH:MM"
function fmtTime(date) {
  return `${String(date.getHours()).padStart(2, '0')}:${String(date.getMinutes()).padStart(2, '0')}`;
}

// Date → "YYYY/MM/DD (曜)"
function fmtDate(date) {
  const y = date.getFullYear();
  const mo = String(date.getMonth() + 1).padStart(2, '0');
  const d = String(date.getDate()).padStart(2, '0');
  const weekdays = ['日', '月', '火', '水', '木', '金', '土'];
  return `${y}/${mo}/${d} (${weekdays[date.getDay()]})`;
}

// "HH:MM" + 今日 → Date（今日の指定時刻）
function shiftStartToDate(hhmm, now) {
  const [h, m] = hhmm.split(':').map(Number);
  const d = new Date(now);
  d.setHours(h, m, 0, 0);
  return d;
}

// 基準時刻Date + 6時間 → 期限Date
function addSixHours(baseDate) {
  return new Date(baseDate.getTime() + 6 * 60 * 60 * 1000);
}
</script>
```

- [ ] **Step 2: DevTest セクションで検証**

上記 `</script>` の直前（つまり同じscript内の末尾）に以下を追加：

```javascript
// --- DevTest --- （検証後に削除）
console.assert(fmtStopwatch(0) === '00:00:00', 'fmtStopwatch 0');
console.assert(fmtStopwatch(59) === '00:00:59', 'fmtStopwatch 59');
console.assert(fmtStopwatch(3661) === '01:01:01', 'fmtStopwatch 3661');
console.assert(fmtMinutes(0) === '0分 (0時間0分)', 'fmtMinutes 0');
console.assert(fmtMinutes(135) === '135分 (2時間15分)', 'fmtMinutes 135');
console.assert(fmtMinutes(180) === '180分 (3時間0分)', 'fmtMinutes 180');
console.assert(fmtRemaining(0) === '0時間0分', 'fmtRemaining 0');
console.assert(fmtRemaining(2 * 3600000 + 15 * 60000) === '2時間15分', 'fmtRemaining 2h15');
console.assert(fmtRemaining(-30 * 60000) === '超過: 0時間30分', 'fmtRemaining over');
console.assert(fmtTime(new Date(2026, 0, 1, 7, 5)) === '07:05', 'fmtTime 07:05');
console.assert(fmtDate(new Date(2026, 3, 24)) === '2026/04/24 (金)', 'fmtDate'); // 2026-04-24は金曜
const sd = shiftStartToDate('07:00', new Date(2026, 3, 24, 15, 30));
console.assert(sd.getHours() === 7 && sd.getMinutes() === 0, 'shiftStartToDate');
const six = addSixHours(sd);
console.assert(six.getHours() === 13 && six.getMinutes() === 0, 'addSixHours');
console.log('All pure function tests passed');
```

- [ ] **Step 3: ブラウザで検証**

`index.html` をリロードし、開発者ツール(Safari: Cmd+Opt+C / Chrome: Cmd+Opt+I)でConsoleを確認。`All pure function tests passed` と出力されること。Errorが出ないこと。

- [ ] **Step 4: DevTest セクションを削除**

`// --- DevTest ---` から `console.log('All pure function tests passed');` までを削除してコミットに含めない形にする。

- [ ] **Step 5: コミット**

```bash
git add index.html
git commit -m "feat: add pure formatting and time calculation functions"
```

---

## Task 5: localStorage 層と初期表示

**Files:**
- Modify: `index.html`（`<script>` 内の末尾に追加）

- [ ] **Step 1: Storage 層を追加**

Pure セクションの後、`</script>` の前に追加：

```javascript
// ============================================================
// --- Storage ---
// localStorage のラッパー。スキーマはspec §5.1 参照。
// ============================================================

const STORAGE_KEY = 'taxi-timer-v1';

function loadState() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) return { shiftStart: '07:00', records: [] };
    const parsed = JSON.parse(raw);
    return {
      shiftStart: parsed.shiftStart || '07:00',
      records: Array.isArray(parsed.records) ? parsed.records : []
    };
  } catch (e) {
    return { shiftStart: '07:00', records: [] };
  }
}

function saveState(state) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify({
    shiftStart: state.shiftStart,
    records: state.records
  }));
}

// ============================================================
// --- State ---
// アプリ実行中の状態。永続値 + ストップウォッチ状態。
// ============================================================

const state = {
  ...loadState(),          // shiftStart, records
  stopwatch: {
    running: false,
    startedAt: null,       // number (ms) or null
    elapsedMs: 0           // 停止時累計
  }
};

function currentStopwatchMs() {
  if (!state.stopwatch.running) return state.stopwatch.elapsedMs;
  return state.stopwatch.elapsedMs + (Date.now() - state.stopwatch.startedAt);
}
```

- [ ] **Step 2: Render セクションの骨組みを追加**

```javascript
// ============================================================
// --- Render ---
// 状態から DOM を更新する。副作用のみ。
// ============================================================

const $ = (id) => document.getElementById(id);

function renderDate() {
  $('today-date').textContent = fmtDate(new Date());
}

function renderShiftStart() {
  $('shift-start-input').value = state.shiftStart;
}

function renderStopwatch() {
  const ms = currentStopwatchMs();
  $('stopwatch-display').textContent = fmtStopwatch(ms / 1000);
}

function renderButtons() {
  const ms = currentStopwatchMs();
  const running = state.stopwatch.running;
  $('btn-start').disabled = running;
  $('btn-record').disabled = !running || ms < 11 * 60 * 1000;
  $('btn-discard').disabled = !running;
}

function renderMetrics() {
  // 合計休憩
  const totalBreakSec = state.records.reduce((s, r) => s + r.durationSec, 0);
  const totalBreakMin = Math.floor(totalBreakSec / 60);
  $('break-total').textContent = fmtMinutes(totalBreakMin);
  $('break-remaining').textContent = fmtMinutes(Math.max(0, 180 - totalBreakMin));

  // 基準時刻
  const now = new Date();
  let baseDate;
  if (state.records.length > 0) {
    baseDate = new Date(state.records[state.records.length - 1].recordedAt);
  } else {
    baseDate = shiftStartToDate(state.shiftStart, now);
  }
  const deadline = addSixHours(baseDate);
  $('deadline-time').textContent = fmtTime(deadline);
  $('deadline-remaining').textContent = fmtRemaining(deadline.getTime() - now.getTime());
}

function renderHistory() {
  const list = $('history-list');
  $('history-count').textContent = state.records.length;
  list.innerHTML = '';
  const sorted = [...state.records].sort(
    (a, b) => new Date(b.recordedAt) - new Date(a.recordedAt)
  );
  for (const r of sorted) {
    const li = document.createElement('li');
    const recDate = new Date(r.recordedAt);
    const mins = Math.floor(r.durationSec / 60);
    const nextDeadline = fmtTime(addSixHours(recDate));
    li.textContent = `${fmtTime(recDate)}  休憩 ${mins}分  →次 ${nextDeadline}`;
    list.appendChild(li);
  }
}

function renderAll() {
  renderDate();
  renderShiftStart();
  renderStopwatch();
  renderButtons();
  renderMetrics();
  renderHistory();
}
```

- [ ] **Step 3: Init セクションを追加**

```javascript
// ============================================================
// --- Init ---
// ============================================================

renderAll();
```

- [ ] **Step 4: ブラウザで検証**

リロード。

- 日付が今日の日付で表示されていること（例: `2026/04/24 (金)`）
- 乗務開始の input が `07:00` になっていること
- 連続走行可能時刻が乗務開始+6時間（例: `13:00`）になっていること
- 連続走行可能時間は現在時刻と `13:00` の差になっていること
- 残り必要休憩が `180分 (3時間0分)`、合計休憩が `0分 (0時間0分)` となっていること
- 記録履歴が `0件` となっていること

- [ ] **Step 5: コミット**

```bash
git add index.html
git commit -m "feat: add storage layer and initial render"
```

---

## Task 6: ストップウォッチのスタート/破棄/記録

**Files:**
- Modify: `index.html`（`<script>` 内に Events セクション追加）

- [ ] **Step 1: Events セクションを追加（Render と Init の間）**

```javascript
// ============================================================
// --- Events ---
// ============================================================

$('btn-start').addEventListener('click', () => {
  state.stopwatch.running = true;
  state.stopwatch.startedAt = Date.now();
  state.stopwatch.elapsedMs = 0;
  renderAll();
});

$('btn-discard').addEventListener('click', () => {
  state.stopwatch.running = false;
  state.stopwatch.startedAt = null;
  state.stopwatch.elapsedMs = 0;
  renderAll();
});

$('btn-record').addEventListener('click', () => {
  const ms = currentStopwatchMs();
  if (ms < 11 * 60 * 1000) return; // 二重防御
  const record = {
    recordedAt: new Date().toISOString(),
    durationSec: Math.floor(ms / 1000)
  };
  state.records.push(record);
  state.stopwatch.running = false;
  state.stopwatch.startedAt = null;
  state.stopwatch.elapsedMs = 0;
  saveState(state);
  renderAll();
});

$('shift-start-input').addEventListener('change', (e) => {
  state.shiftStart = e.target.value || '07:00';
  saveState(state);
  renderAll();
});

$('btn-reset').addEventListener('click', () => {
  if (!confirm('記録履歴を全て削除します。よろしいですか？')) return;
  state.records = [];
  state.stopwatch.running = false;
  state.stopwatch.startedAt = null;
  state.stopwatch.elapsedMs = 0;
  saveState(state);
  renderAll();
});
```

- [ ] **Step 2: ブラウザで検証（スタート〜破棄）**

1. 「スタート」を押す → 記録・破棄ボタンが有効化、スタートは無効化
2. ストップウォッチは動かないが記録履歴は増えないこと（後のタスクで動くようになる）
3. 「破棄」を押す → 全ボタン初期状態、`00:00:00` に戻る

- [ ] **Step 3: コミット**

```bash
git add index.html
git commit -m "feat: wire stopwatch start/discard/record/reset buttons"
```

---

## Task 7: 1秒ティック（リアルタイム更新）

**Files:**
- Modify: `index.html`（Tick セクション追加）

- [ ] **Step 1: Tick セクションを追加（Events の後）**

```javascript
// ============================================================
// --- Tick ---
// 1秒ごとに時計・メトリクス・ボタン状態を再描画する。
// ============================================================

setInterval(() => {
  renderStopwatch();
  renderMetrics();
  renderButtons();
}, 1000);
```

- [ ] **Step 2: ブラウザで検証**

1. 連続走行可能時間が1秒ごとに減っていくこと（例: `2時間15分` → `2時間14分` と分が変わる）
2. 「スタート」を押す → ストップウォッチが1秒ごとに進むこと（`00:00:01`, `00:00:02`...）
3. そのまま待つと「記録」ボタンが有効化される時刻（11分経過）まで待つと煩雑なため、検証のため一時的に閾値を10秒に変更：`$('btn-record').disabled = !running || ms < 10 * 1000;`
4. 10秒後に記録ボタンが有効化されること確認
5. 「記録」を押す → 履歴に1件追加、ストップウォッチリセット、残り必要休憩と合計休憩が更新されること
6. 履歴項目が `HH:MM  休憩 0分  →次 HH:MM` 形式になっていること
7. 検証が終わったら閾値を `11 * 60 * 1000` に戻す

- [ ] **Step 3: ページをリロードして永続化を検証**

リロード後も記録が残っていること、乗務開始時刻が保持されていること。

- [ ] **Step 4: 「全リセット」を検証**

「全リセット」を押して履歴が消えること、確認ダイアログが出ること。

- [ ] **Step 5: コミット**

```bash
git add index.html
git commit -m "feat: add 1-second tick for realtime updates"
```

---

## Task 8: 視覚的な注意喚起（警告色・超過表示）

**Files:**
- Modify: `index.html`（`renderMetrics` に色制御追加）

- [ ] **Step 1: renderMetrics を以下に差し替え**

既存の `renderMetrics` を以下で置き換え：

```javascript
function renderMetrics() {
  // 合計休憩
  const totalBreakSec = state.records.reduce((s, r) => s + r.durationSec, 0);
  const totalBreakMin = Math.floor(totalBreakSec / 60);
  $('break-total').textContent = fmtMinutes(totalBreakMin);
  $('break-remaining').textContent = fmtMinutes(Math.max(0, 180 - totalBreakMin));

  // 基準時刻
  const now = new Date();
  let baseDate;
  if (state.records.length > 0) {
    baseDate = new Date(state.records[state.records.length - 1].recordedAt);
  } else {
    baseDate = shiftStartToDate(state.shiftStart, now);
  }
  const deadline = addSixHours(baseDate);
  $('deadline-time').textContent = fmtTime(deadline);

  const remainingMs = deadline.getTime() - now.getTime();
  const remainingEl = $('deadline-remaining');
  remainingEl.textContent = fmtRemaining(remainingMs);

  const metricsSection = document.querySelector('section.metrics');
  remainingEl.classList.remove('warn', 'danger');
  metricsSection.classList.remove('over');
  if (remainingMs < 0) {
    metricsSection.classList.add('over');
    remainingEl.classList.add('danger');
  } else if (remainingMs <= 30 * 60 * 1000) {
    remainingEl.classList.add('danger');
  } else if (remainingMs <= 60 * 60 * 1000) {
    remainingEl.classList.add('warn');
  }
}
```

- [ ] **Step 2: ブラウザで検証**

1. 乗務開始時刻を「現在時刻 + 1時間」に設定 → 連続走行可能時間が通常色
2. 乗務開始時刻を「現在時刻 + 30分」に設定 → オレンジ（warn）
3. 乗務開始時刻を「現在時刻 + 15分」に設定 → 赤（danger）
4. 乗務開始時刻を「現在時刻 - 10分」に設定 → 赤背景（over）＋「超過: 0時間10分」と表示
5. 検証後、乗務開始時刻を `07:00` に戻す

- [ ] **Step 3: コミット**

```bash
git add index.html
git commit -m "feat: add visual warnings for approaching/exceeded deadline"
```

---

## Task 9: apple-touch-icon と PWA メタタグ

**Files:**
- Create: `icon-180.png`
- Modify: `index.html`（`<head>` にメタタグ追加）

- [ ] **Step 1: 暫定アイコンを生成**

Macのターミナルで以下を実行してプロジェクトディレクトリに180x180pxの単色アイコンを作る：

```bash
cd "/Users/hideakimacbookair/Library/Mobile Documents/com~apple~CloudDocs/タクシー乗務タイマー"
sips -z 180 180 -s format png --out icon-180.png /System/Library/CoreServices/DefaultDesktop.heic 2>/dev/null || python3 -c "
from PIL import Image, ImageDraw, ImageFont
img = Image.new('RGB', (180, 180), '#3a7bd5')
d = ImageDraw.Draw(img)
d.text((60, 60), 'TT', fill='white')
img.save('icon-180.png')
" 2>/dev/null || curl -sL 'https://via.placeholder.com/180/3a7bd5/ffffff?text=TT' -o icon-180.png
```

**注:** 上記のどれかで失敗する場合、ChromeまたはSafariでMacPaintやPreview.appで180x180pxのPNGを手動作成して保存する。後日アイコンを差し替える予定（仕様 §8 参照）。

- [ ] **Step 2: `<head>` にメタタグを追加**

`<title>` の直後に追加：

```html
<link rel="apple-touch-icon" sizes="180x180" href="icon-180.png">
<meta name="apple-mobile-web-app-title" content="乗務タイマー">
<meta name="theme-color" content="#1a1a1a">
```

- [ ] **Step 3: ブラウザで検証**

- Safari開発者ツールでnetwork tabを見て `icon-180.png` が200で取得されていること
- `<head>` の内容を確認

- [ ] **Step 4: コミット**

```bash
git add icon-180.png index.html
git commit -m "feat: add apple-touch-icon and PWA meta tags"
```

---

## Task 10: GitHub リポジトリ作成と初回プッシュ

**Files:**
- なし（GitHub側操作 + git remote）

- [ ] **Step 1: GitHubで新規リポジトリ作成**

ブラウザで https://github.com/new を開く。以下の設定で作成：
- Repository name: `taxi-timer`
- Public
- README・.gitignore・LICENSE は追加しない（既にローカルにある）
- 「Create repository」

- [ ] **Step 2: リモートを追加してプッシュ**

```bash
cd "/Users/hideakimacbookair/Library/Mobile Documents/com~apple~CloudDocs/タクシー乗務タイマー"
git branch -M main
git remote add origin https://github.com/hidenaka/taxi-timer.git
git push -u origin main
```

認証が必要な場合はブラウザでGitHubログイン、またはPersonal Access Tokenを使う。

- [ ] **Step 3: プッシュ結果を確認**

ブラウザで `https://github.com/hidenaka/taxi-timer` を開き、`index.html` 等のファイルが表示されていること。

---

## Task 11: GitHub Pages 有効化とホーム画面追加

**Files:**
- なし（GitHub側設定 + iPad操作）

- [ ] **Step 1: GitHub Pages を有効化**

1. ブラウザで `https://github.com/hidenaka/taxi-timer/settings/pages` を開く
2. 「Build and deployment」の「Source」で **Deploy from a branch** を選択
3. 「Branch」で `main` / `/ (root)` を選択
4. 「Save」を押す
5. 数秒〜数分後、ページ上部に `Your site is live at https://hidenaka.github.io/taxi-timer/` と表示される

- [ ] **Step 2: PC Safariで動作確認**

`https://hidenaka.github.io/taxi-timer/` を開き、ローカルと同じ動作をすること。localStorageは localhost と github.io で別管理。

- [ ] **Step 3: iPad mini Safari で動作確認**

1. iPad mini Safariで `https://hidenaka.github.io/taxi-timer/` を開く
2. 全機能動作確認：スタート、11分待たずに検証する場合は再度閾値を一時変更してテスト
3. 共有ボタン → 「ホーム画面に追加」 → タイトル「乗務タイマー」で追加
4. ホーム画面のアイコンをタップして起動すること確認

- [ ] **Step 4: 完了**

プランの全タスク完了。

---

## 自己レビューチェックリスト

設計書(spec §9) の完成定義と照合：

- [x] GitHub Pages で公開され、URL にアクセスして iPad Safari で動作する（Task 11）
- [x] ストップウォッチが正しく動作する（Task 6-7）
- [x] 11分閾値が正しく機能する（Task 5, 6）
- [x] 連続走行可能時間が1秒ごとにリアルタイム更新される（Task 7）
- [x] 記録履歴が新しい順に表示される（Task 5 renderHistory）
- [x] ページリロード後も記録が復元される（Task 5 Storage層）
- [x] 乗務開始時刻がデフォルト 7:00 で、編集可能・保存される（Task 2, 5, 6）

spec §2 の機能要件：
- 2.1 ストップウォッチの全ボタン挙動 → Task 5, 6 でカバー
- 2.2 乗務開始時刻入力・永続化 → Task 5, 6 でカバー
- 2.3 主要表示項目8つ → Task 5 renderAll でカバー
- 2.4 操作（スタート/記録/破棄/全リセット）→ Task 6 でカバー
- 6.3 視覚的注意喚起（残り30分赤・超過）→ Task 8 でカバー
- 7 デプロイ → Task 10, 11 でカバー
