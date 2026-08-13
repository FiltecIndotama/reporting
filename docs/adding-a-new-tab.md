# Training Manual — Adding a New Tab to the Filtec Sales Dashboard

**Worked example: adding a `Q&A` tab (Tab 9)**

Audience: anyone editing the dashboard who can read HTML but has not worked on
this file before. No build tools, no framework, no npm. If you can edit a text
file and commit it, you can add a tab.

Reference commit: `6f62901` — "Add Staff tab" — is the most recent real example
of this exact procedure. When in doubt, run `git show 6f62901` and copy what it did.

---

## 1. What you are editing

The entire website is **one file**:

```
filtec-sales-dashboard.html      (~2,426 lines)
```

Everything lives inside it — HTML, CSS (in a single `<style>` block near the top),
and JavaScript (in a single `<script>` block at the bottom). There is no build
step. What you commit is what the browser runs.

Two external scripts are pulled in at the end (`/view-counter.js`,
`/sw-register.js`). You never need to touch them to add a tab.

### The file's five regions

| Lines (approx.) | Region | What's there |
|---|---|---|
| 1–11 | `<head>` | Title, icons, Google Fonts |
| 12–214 | `<style>` | All CSS. Design tokens at `:root`, then components |
| 216–258 | `.hero` | Language buttons, ⋮ menu, breadcrumb, title |
| 260–269 | `.tabs-bar` | **The row of tab buttons — edit point 1** |
| 271–2304 | `.content` | **One `.page-panel` div per tab — edit point 2** |
| 2314–2420 | `<script>` | `setPage()`, `setLang()`, toggles — **edit point 3** |

> Line numbers drift every time content is added. Always locate regions by
> searching for the marker text (e.g. search for `tabs-bar`, or `id="page-8"`),
> never by jumping to a memorised line number.

---

## 2. The mental model — how a tab actually works

There is no router and no JavaScript framework. Tabs are **CSS visibility, nothing more.**

Every tab is two pieces of HTML that share a number:

```
<button class="tab-btn" id="tab-8" onclick="setPage(8)">    ← the button
<div    class="page-panel"  id="page-8">                    ← the content
```

All panels are always in the DOM. Only the active one is visible, because of
two CSS rules:

```css
.page-panel        { display: none;  }
.page-panel.active { display: block; }
```

And one JavaScript function that moves the `active` class around:

```js
function setPage(n) {
  [1, 2, 3, 4, 5, 6, 7, 8].forEach(function(i) {
    document.getElementById('page-' + i).classList.toggle('active', i === n);
    document.getElementById('tab-'  + i).classList.toggle('active', i === n);
  });
  window.scrollTo({ top: document.querySelector('.tabs-bar').offsetTop, behavior: 'smooth' });
}
```

**This is the single most important thing to understand:** that array
`[1, 2, 3, 4, 5, 6, 7, 8]` is hard-coded. If you add `page-9` and `tab-9` but
forget to add `9` to that array, your tab button will do **nothing at all** when
clicked — no error, no console message, just a dead button. This is the #1
mistake when adding a tab.

---

## 3. The four edit points — checklist

Adding any tab is always these four edits, in this order:

- [ ] **1. Tab button** — add one `<button>` to `.tabs-bar`
- [ ] **2. Content panel** — add one `<div class="page-panel" id="page-N">` inside `.content`
- [ ] **3. Register the number** — add `N` to the array in `setPage()`
- [ ] **4. Any new toggles/CSS** — only if your tab uses a collapsible table or a new component

Edits 1–3 are mandatory for every tab. Edit 4 depends on your content.

---

## 4. Step-by-step — building the Q&A tab

Our new tab is number **9** (there are 8 today: Brand, Key Accounts, GP %,
Retention, Segmentation, Expenses, Net Profit, Staff).

### Step 4.1 — Add the tab button

Find the `.tabs-bar` block (search for `class="tabs-bar"`). It ends with the
Staff button. Add your button as the **last line before `</div>`**:

```html
  <button class="tab-btn" id="tab-8" onclick="setPage(8)"><span class="en">Staff</span><span class="zh">销售人员</span><span class="id">Staf</span></button>
  <button class="tab-btn" id="tab-9" onclick="setPage(9)"><span class="en">Q&amp;A</span><span class="zh">问答</span><span class="id">Tanya Jawab</span></button>
</div>
```

Three things to note:

- **No `active` class.** Only `tab-1` (Brand) carries `active`, because Brand is
  the tab shown on load. Never add a second `active`.
- **`&amp;` not `&`.** In HTML, a bare `&` is the start of an entity. Writing
  `Q&A` will usually still render, but `Q&amp;A` is correct and is what the rest
  of the file does (see `Boll &amp; Kirch`).
- **Three languages, always.** See section 5.
- **Keep the label short.** The bar is `flex-wrap: wrap` and `position: sticky`,
  so long labels wrap it onto an extra row that then sits pinned to the top of
  the screen on every scroll. One or two words — "Q&A", "Staff", "GP %".

### Step 4.2 — Add the content panel

Scroll to the very end of the `.content` region. You are looking for the closing
of `page-8`, which looks like this (around line 2298):

```html
      </ul>
    </div>          ← closes the last insight-card of page-8
  </div>            ← closes  <div class="page-panel" id="page-8">

  <script src="/view-counter.js"></script>

</div>              ← closes  <div class="content">
```

Insert your whole new panel **between the close of `page-8` and the
`view-counter.js` script tag**. The skeleton:

```html
  <!-- ============ TAB 9 : Q&A ============ -->
  <div class="page-panel" id="page-9">
    <div class="page-head">
      <span class="en">Questions &amp; Answers</span><span class="zh">问答</span><span class="id">Tanya Jawab</span>
    </div>

    <p>
      <span class="en">Common questions about how the numbers on this dashboard are built, where they come from, and what they do and do not prove.</span>
      <span class="zh">关于本仪表板数据如何生成、来源为何、以及能证明与不能证明什么的常见问题。</span>
      <span class="id">Pertanyaan umum tentang bagaimana angka di dasbor ini dibangun, dari mana asalnya, dan apa yang dibuktikan maupun tidak.</span>
    </p>

    <div class="chart-card">
      <div class="qa-list">
        <!-- Q&A items go here -->
      </div>
    </div>
  </div>
```

Note the comment banner. Every tab in this file has one
(`<!-- ============ TAB 1 : Brand ============ -->`) — it is the only navigation
aid in a 2,400-line file, so keep the convention.

### Step 4.3 — Register the tab number in `setPage()`

Find `function setPage(n)` near the bottom. Change the array:

```js
function setPage(n) {
  [1, 2, 3, 4, 5, 6, 7, 8, 9].forEach(function(i) {
    //                     ^^^ add your number here
```

**Do not skip this step.** Without it the button is dead.

### Step 4.4 — Add the Q&A component

A Q&A tab is mostly text, and the file has no accordion component yet, so this
one tab needs a small amount of new CSS and one new JS function. (Most tabs need
neither — they reuse the components in section 6.)

**CSS** — add at the end of the `<style>` block, just before `footer { ... }`.
Keep to the existing design tokens (`var(--card)`, `var(--border)`, `var(--blue)`)
so it matches automatically:

```css
.qa-list { display:flex; flex-direction:column; gap:10px; }
.qa-item { background:rgba(255,255,255,0.03); border:1px solid var(--border); border-radius:10px; overflow:hidden; }
.qa-q { display:flex; align-items:flex-start; gap:10px; width:100%; background:none; border:none; color:#fff; text-align:left; padding:14px 16px; font-size:0.9rem; font-weight:700; line-height:1.45; cursor:pointer; font-family:'Source Serif 4',serif; }
.qa-q::after { content:'+'; margin-left:auto; color:var(--blue); font-size:1.15rem; font-weight:800; flex:none; line-height:1; }
.qa-item.open .qa-q::after { content:'−'; }
.qa-q:hover { background:rgba(255,255,255,0.04); }
.qa-a { display:none; padding:0 16px 16px; font-size:0.88rem; color:var(--text); line-height:1.65; }
.qa-item.open .qa-a { display:block; }
.qa-a p + p { margin-top:10px; }
.qa-tag { display:inline-block; font-size:0.66rem; font-weight:700; padding:2px 8px; border-radius:10px; margin-bottom:10px; background:rgba(74,108,247,0.18); color:var(--blue); }
```

**JavaScript** — add one function to the `<script>` block, next to the other
toggles:

```js
function toggleQa(btn){ btn.parentElement.classList.toggle('open'); }
```

The existing `toggleXxxTable()` functions each hard-code one element ID, which
works when there are eight tables but not when there are twenty questions. This
version takes the clicked button as an argument and toggles its parent, so **one
function serves every Q&A item** — no new JS when you add question twenty-one.

**One Q&A item** — repeat this block per question, inside `.qa-list`:

```html
<div class="qa-item">
  <button class="qa-q" onclick="toggleQa(this)">
    <span class="en">Why don't the revenue totals match between tabs?</span>
    <span class="zh">为什么各标签页的收入总额对不上？</span>
    <span class="id">Mengapa total pendapatan berbeda antar tab?</span>
  </button>
  <div class="qa-a">
    <span class="qa-tag"><span class="en">Data</span><span class="zh">数据</span><span class="id">Data</span></span>
    <p>
      <span class="en">Because the tabs deliberately measure different things. The Segmentation tab uses Sales Invoice <code>grand_total</code>, which includes ~11% PPN/VAT. The Brand tab (Rp 6,171,563,874) and GP % tab (Rp 6,139,939,624) are net of tax. The ~11% gap is the expected tax difference, not an error.</span>
      <span class="zh">因为各标签页衡量的口径不同。客户细分标签使用销售发票的<code>grand_total</code>，其中包含约11%的PPN增值税；品牌标签（Rp 6,171,563,874）和毛利率标签（Rp 6,139,939,624）为不含税金额。约11%的差额是正常的税项差异，并非错误。</span>
      <span class="id">Karena tiap tab sengaja mengukur hal yang berbeda. Tab Segmentasi memakai <code>grand_total</code> Sales Invoice yang termasuk PPN ~11%. Tab Merek (Rp 6.171.563.874) dan tab GP % bersih dari pajak. Selisih ~11% adalah perbedaan pajak yang wajar, bukan kesalahan.</span>
    </p>
  </div>
</div>
```

That is the complete tab. Four edits, done.

---

## 5. The trilingual rule — non-negotiable

**Every single string a user can read must exist in English, Chinese and Bahasa
Indonesia.** This is the rule most easily broken and the hardest to notice,
because a missing translation looks perfect in English and leaves a **blank gap**
for a Chinese or Indonesian reader.

The mechanism is pure CSS. All three versions are always in the HTML; the body
class decides which is visible:

```css
.zh { display: none; }              /* hidden by default        */
.id { display: none; }
body.lang-zh .zh { display: inline; }   /* shown when 中文 picked   */
body.lang-zh .en { display: none;   }
body.lang-id .id { display: inline; }
body.lang-id .en { display: none;   }
```

`setLang()` sets `document.body.className` to `''`, `lang-zh` or `lang-id`.

### The pattern

```html
<span class="en">English text</span><span class="zh">中文文本</span><span class="id">Teks Bahasa</span>
```

### Inline vs block

| Use | When | Shown as |
|---|---|---|
| `.en` / `.zh` / `.id` | Inside a sentence, label, cell, heading | `inline` |
| `.en-block` / `.zh-block` / `.id-block` | A whole paragraph or block element | `block` |

Use the `-block` variants only when the translated content is itself a block
(e.g. a whole `<div>`); for ordinary text — including table cells and headings —
the plain classes are correct.

### Rules of thumb

- **Never leave a bare untranslated string.** The only exceptions in the file are
  proper nouns that are identical in all three languages (`Boll & Kirch`, staff
  names, `GP %`, currency figures) — those sit outside the spans deliberately.
- **Keep the three spans adjacent with no whitespace between them.** Writing them
  on separate lines injects spaces into the rendered sentence.
- **Numbers do not get translated, but their punctuation does.** English/Chinese
  use `Rp 6,171,563,874`; Indonesian uses `Rp 6.171.563.874` (dot separators).
  The existing tabs follow this — match it.
- **Test all three.** Click 🇬🇧 / 🇨🇳 / 🇮🇩 before committing. See section 8.

---

## 6. Component library — reuse, don't invent

Before writing new CSS, check this table. Almost everything you need already
exists, and reusing it is what keeps the eight tabs looking like one product.

| Component | Class | Use for |
|---|---|---|
| Section heading | `.page-head` | Tab title, "Chart 9: …", "Table 1320: …" |
| Card container | `.chart-card` | Wraps a chart + its table + notes |
| Warning box | `.caveat` | Data-quality caveats (⚠️ + text). Used heavily — be honest about limits |
| KPI tiles | `.stat-row` > `.stat-mini` (`.sv` value, `.sl` label, `.sb` sub-label) | Headline numbers at the top of a tab |
| Bar chart | `.chart` > `.bar-row` > `.bar-label-row` + `.track` > `.fill` | The house chart. Widths are literal percentages in `style="width:NN%"` |
| Bar colours | `.fill.s1` blue, `.s2` orange, `.s3` grey, `.s4` red; `.o1`–`.o4` blue ramp | Series colours |
| Chart legend | `.legend` > `.legend-item` > `.swatch` | Above a chart |
| Axis note | `.baseline-axis` | "0 — scaled to Rp X (Name) · date range" |
| Data table | `<table id="…">` + `.toggle-btn` | Hidden by default; `.show` reveals it |
| Wide table | `.so-scroll` > `table.so-wide` | Many columns; scrolls with sticky header |
| Numeric cell | `td.num` | Right-aligned, tabular figures |
| Name cell | `td.name` | Bold white first column |
| Recommendations | `.insight-card` > `.insight-list` > `li` > `.insight-num` | The numbered "what to do about it" list that closes every tab |
| Positive / negative | `.pos` / `.neg` | Green / red figures |
| Confidence pill | `.cf` + `.cf-h` / `.cf-m` / `.cf-l` / `.cf-n` | High / medium / low / none confidence badges |
| Footnote | `.foot-note` | Source line under a table |

### Design tokens

Never hard-code a hex colour that already has a token:

```css
--blue: #4a6cf7;   --accent: #f4813f;   --bg: #0d0f1a;    --card: #151826;
--text: #e8eaf0;   --muted: #8b8fa8;    --border: rgba(255,255,255,0.08);
--gold: #f59e0b;   --red: #f85149;      --green: #3fb950;
--series-1: #3987e5;  --series-2: #d95926;
```

Font family for any new button: `'Source Serif 4', serif` — buttons don't inherit
it, which is why every button rule in the file restates it.

### Charts are static HTML

There is no charting library. A bar is a `div` with an inline percentage width,
computed by hand relative to the largest value:

```html
<div class="bar-row">
  <div class="bar-label-row"><span class="bar-name">Dedeng Suryaman</span><span class="bar-value">Rp 3,584,174,562 · 17.5%</span></div>
  <div class="track"><div class="fill s1" style="width:100.0%">Rp 3.58B</div></div>
</div>
```

The largest bar gets `width:100.0%`; the rest are `value / max × 100`. Give
zero-value bars `width:0.5%` so the row is still visible (see the Staff tab).

---

## 7. Numbering conventions

The dashboard numbers its charts and tables **globally, not per tab**, so figures
can be cited unambiguously across tabs ("see Table 1314").

| Series | Last used | Your Q&A tab starts at |
|---|---|---|
| Charts | Chart 11 (Staff) | Chart 12 |
| Tables | Table 1322 (Staff) | Table 1323 |
| Tabs / pages | Tab 8 (Staff) | Tab 9 |

A Q&A tab typically has no charts or tables at all — if yours does, use the next
number in sequence and never reuse one.

---

## 8. Testing before you commit

No test suite exists, so this checklist **is** the test suite. Open the file
directly in a browser (`file://` works fine — nothing needs a server).

- [ ] **The new tab button appears** in the tab bar, at the end of the row.
- [ ] **Clicking it shows your panel** and hides the previous one. *(If nothing
      happens → you forgot step 4.3.)*
- [ ] **Clicking every other tab still works** — 1 through 9, in order. A stray
      unclosed `</div>` in your panel silently swallows the tabs after it.
- [ ] **Only one tab is highlighted at a time** (blue). Two highlighted = a
      duplicate `active` class.
- [ ] **Brand (Tab 1) is still the tab shown on page load.**
- [ ] **Switch to 🇨🇳 中文** — read your whole tab. No English left behind, no
      blank gaps.
- [ ] **Switch to 🇮🇩 Bahasa** — same check.
- [ ] **Switch back to 🇬🇧 English** — no Chinese or Indonesian leaking through.
- [ ] **Q&A accordions open and close**, and the `+` flips to `−`.
- [ ] **Narrow the window to phone width (~390px).** The tab bar is
      `flex-wrap: wrap`, so a 9th button pushes it onto more rows. Because the bar
      is also `position: sticky; top: 0`, every extra row permanently eats
      vertical space on phones — check it still leaves room to read the content.
      Nothing should overflow horizontally.
- [ ] **Open the browser console** (F12) — no red errors.

Quick structural sanity check from the terminal:

```bash
# Should print 9 tab buttons and 9 page panels — the two counts must match
grep -c 'class="tab-btn' filtec-sales-dashboard.html
grep -c 'class="page-panel' filtec-sales-dashboard.html
```

---

## 9. Committing and deploying

Work on a branch, never directly on `main`:

```bash
git checkout -b claude/add-qa-tab
git add filtec-sales-dashboard.html
git commit -m "Add Q&A tab: common questions on data sources and methodology"
git push -u origin claude/add-qa-tab
```

Then open a pull request against `main`. The site deploys from `main`
(paulsworld.vercel.app), so merging is what publishes the change.

### Commit message style

Follow the existing convention — a one-line summary, then a body explaining what
was added and, crucially, *what the data does and does not prove*:

```
Add Staff tab: sales-order attribution by sales staff (#1)

Adds an eighth tab, Staff, to filtec-sales-dashboard.html covering sales-staff
attribution across all 230 Sales Orders: quotation ownership per staff (recorded
fact), derived per-order attribution with confidence bands (indicative), and
remediation steps for ERPNext.

Adds Chart 9, Chart 10, Chart 11 and Tables 1320, 1321, 1322 in the dashboard's
existing trilingual idiom.
```

---

## 10. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Tab button does nothing when clicked | `9` missing from the `setPage()` array | Step 4.3 |
| Button works but the panel is blank | `id="page-9"` typo, or the panel was pasted outside `.content` | Check the id spells exactly `page-9`; confirm it sits before `</div>` closing `.content` |
| Two tabs highlighted at once | You copied `class="tab-btn active"` | Remove `active` — only `tab-1` has it |
| Your tab shows on load, stacked under Brand | You copied `class="page-panel active"` | Remove `active` — only `page-1` has it |
| Tabs after yours stopped working | Unbalanced `<div>` in your panel | Count opening vs closing `</div>`; your panel must close exactly once |
| Text disappears when switching language | Missing `.zh` or `.id` span | Add the translation — section 5 |
| Sentence has odd extra spaces | Newlines between the `.en`/`.zh`/`.id` spans | Put the three spans adjacent, no whitespace between |
| Accordion won't open | `onclick="toggleQa(this)"` missing the `this` | Pass `this` — the function needs the clicked element |
| New button uses the wrong font | Buttons don't inherit `font-family` | Add `font-family:'Source Serif 4',serif` to the rule |
| Chart bars all the same length | Widths not scaled to the max value | Recompute: `value / max × 100` |

---

## 11. Optional improvements (not required)

Worth knowing about, but skip these for a first tab:

- **Deep links.** `setPage()` doesn't touch the URL, so you can't link straight
  to a tab. Reading `location.hash` on load and writing it in `setPage()` would
  fix it — a change to shared machinery, so do it as its own commit.
- **Page `<title>`.** It's fixed at "Filtec Sales Dashboard — Brand" and doesn't
  follow the active tab.
- **File size.** At ~273KB and 2,426 lines the single-file approach is nearing
  its comfortable limit. Splitting per-tab HTML would be a significant refactor —
  worth discussing before tab 12 or so, not something to do while adding a tab.

---

## Appendix — copy-paste starter for the Q&A tab

Everything from section 4 in one block. Paste the panel after `page-8` closes,
add the button to `.tabs-bar`, add `9` to `setPage()`, add the CSS before
`footer`, and add `toggleQa` to the script block.

```html
<!-- ============ TAB 9 : Q&A ============ -->
<div class="page-panel" id="page-9">
  <div class="page-head">
    <span class="en">Questions &amp; Answers</span><span class="zh">问答</span><span class="id">Tanya Jawab</span>
  </div>

  <p>
    <span class="en">Common questions about how the numbers on this dashboard are built, where they come from, and what they do and do not prove.</span>
    <span class="zh">关于本仪表板数据如何生成、来源为何、以及能证明与不能证明什么的常见问题。</span>
    <span class="id">Pertanyaan umum tentang bagaimana angka di dasbor ini dibangun, dari mana asalnya, dan apa yang dibuktikan maupun tidak.</span>
  </p>

  <div class="chart-card">
    <div class="qa-list">

      <div class="qa-item">
        <button class="qa-q" onclick="toggleQa(this)">
          <span class="en">Where does this data come from?</span>
          <span class="zh">这些数据来自哪里？</span>
          <span class="id">Dari mana data ini berasal?</span>
        </button>
        <div class="qa-a">
          <span class="qa-tag"><span class="en">Source</span><span class="zh">来源</span><span class="id">Sumber</span></span>
          <p>
            <span class="en">ERPNext (Filtec), from submitted Sales Invoices, Sales Orders and Quotations across both companies — PT Filtec Indotama Teknindo and PT Filtec Indo Teknologi. Each tab states its own date window and record count in the note under its charts.</span>
            <span class="zh">来自ERPNext（Filtec）系统中两家公司——PT Filtec Indotama Teknindo 与 PT Filtec Indo Teknologi——已提交的销售发票、销售订单与报价单。每个标签页的图表下方均注明其数据时间范围与记录数量。</span>
            <span class="id">Dari ERPNext (Filtec), berdasarkan Faktur Penjualan, Sales Order, dan Quotation yang sudah disubmit di kedua perusahaan — PT Filtec Indotama Teknindo dan PT Filtec Indo Teknologi. Tiap tab mencantumkan rentang tanggal dan jumlah record-nya di catatan bawah grafik.</span>
          </p>
        </div>
      </div>

      <div class="qa-item">
        <button class="qa-q" onclick="toggleQa(this)">
          <span class="en">Why don't the revenue totals match between tabs?</span>
          <span class="zh">为什么各标签页的收入总额对不上？</span>
          <span class="id">Mengapa total pendapatan berbeda antar tab?</span>
        </button>
        <div class="qa-a">
          <span class="qa-tag"><span class="en">Data</span><span class="zh">数据</span><span class="id">Data</span></span>
          <p>
            <span class="en">Because the tabs deliberately measure different things. The Segmentation tab uses Sales Invoice <code>grand_total</code>, which includes ~11% PPN/VAT. The Brand tab (Rp 6,171,563,874) and GP % tab (Rp 6,139,939,624) are net of tax. The ~11% gap is the expected tax difference, not an error.</span>
            <span class="zh">因为各标签页衡量的口径不同。客户细分标签使用销售发票的<code>grand_total</code>，其中包含约11%的PPN增值税；品牌标签（Rp 6,171,563,874）和毛利率标签（Rp 6,139,939,624）为不含税金额。约11%的差额是正常的税项差异，并非错误。</span>
            <span class="id">Karena tiap tab sengaja mengukur hal yang berbeda. Tab Segmentasi memakai <code>grand_total</code> Sales Invoice yang termasuk PPN ~11%. Tab Merek (Rp 6.171.563.874) dan tab GP % bersih dari pajak. Selisih ~11% adalah perbedaan pajak yang wajar, bukan kesalahan.</span>
          </p>
        </div>
      </div>

      <div class="qa-item">
        <button class="qa-q" onclick="toggleQa(this)">
          <span class="en">How often is the dashboard updated?</span>
          <span class="zh">仪表板多久更新一次？</span>
          <span class="id">Seberapa sering dasbor diperbarui?</span>
        </button>
        <div class="qa-a">
          <span class="qa-tag"><span class="en">Process</span><span class="zh">流程</span><span class="id">Proses</span></span>
          <p>
            <span class="en">The figures are a snapshot, not a live feed. They are refreshed by pulling from ERPNext and updating this page, so the date range printed on each tab is the authoritative "as of" — always read it before quoting a number.</span>
            <span class="zh">页面数据为快照，并非实时数据流。更新方式是从ERPNext提取数据后更新本页面，因此每个标签页上标注的日期范围即为准确的"截至"时点——引用数字前请务必先查看。</span>
            <span class="id">Angka di sini adalah snapshot, bukan data langsung. Diperbarui dengan menarik data dari ERPNext lalu memperbarui halaman ini, jadi rentang tanggal di tiap tab adalah acuan "per tanggal" — selalu baca dulu sebelum mengutip angka.</span>
          </p>
        </div>
      </div>

    </div>
  </div>
</div>
```
