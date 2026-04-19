![第03章: Development Workflows](images/chapter-header.png)

> **AI が、こちらがまだ言語化できていない bug まで見つけてくれるとしたらどうでしょうか？**

この章では、GitHub Copilot CLI を日々の開発 workflow の中で使っていきます。testing、refactoring、debugging、Git といった、普段すでに使っている流れの中に Copilot CLI を組み込みます。

## 🎯 学習目標

この章を終えると、次のことができるようになります。

- Copilot CLI を使って包括的な code review を行う
- legacy code を安全に refactor する
- AI の助けを借りて issue を debug する
- test を自動生成する
- git workflow に Copilot CLI を統合する

> ⏱️ **想定時間**: 約60分（読む時間 15分 + ハンズオン 45分）

---

## 🧩 現実世界のたとえ: 大工さんの Workflow

大工さんは単に tool の使い方を知っているだけではなく、仕事ごとに workflow を持っています。

<img src="images/carpenter-workflow-steps.png" alt="Craftsman workshop showing three workflow lanes: Building Furniture (Measure, Cut, Assemble, Finish), Fixing Damage (Assess, Remove, Repair, Match), and Quality Check (Inspect, Test Joints, Check Alignment)" width="800"/>

開発者も同じです。task ごとに流れがあり、GitHub Copilot CLI はその workflow をより効率的で実用的なものにしてくれます。

---

# 5 つの Workflows

<img src="images/five-workflows.png" alt="Five glowing neon icons representing code review, testing, debugging, refactoring, and git integration workflows" width="800"/>

以下の workflow はそれぞれ独立しています。今必要なものだけを選んで読んでも、順番に全部試しても大丈夫です。

---

## 自分に合うところから進めよう

この章では、開発者がよく使う 5 つの workflow を扱います。**ただし、一度に全部読む必要はありません。** 各 workflow は下の折りたたみ section にまとまっています。自分の project や今の関心に合うものから試してください。あとで戻って他を読むこともできます。

<img src="images/five-workflows-swimlane.png" alt="Five Development Workflows: Code Review, Refactoring, Debugging, Test Generation, and Git Integration shown as horizontal swimlanes" width="800"/>

| やりたいこと | 該当セクション |
|---|---|
| merge 前に code review したい | [Workflow 1: Code Review](#workflow-1-code-review) |
| 散らかった code や legacy code を整理したい | [Workflow 2: Refactoring](#workflow-2-refactoring) |
| bug の原因を追って修正したい | [Workflow 3: Debugging](#workflow-3-debugging) |
| code の test を生成したい | [Workflow 4: Test Generation](#workflow-4-test-generation) |
| commit や PR をもっとよくしたい | [Workflow 5: Git Integration](#workflow-5-git-integration) |
| 実装前に調べ物をしたい | [Quick Tip: Research Before You Plan or Code](#quick-tip-research-before-you-plan-or-code) |
| bug 修正の一連の流れを見たい | [Putting It All Together](#putting-it-all-together-bug-fix-workflow) |

**下の workflow を開いて**、GitHub Copilot CLI がその作業をどう助けるかを見てみましょう。

---

<a id="workflow-1-code-review"></a>
<details>
<summary><strong>Workflow 1: Code Review</strong> - files を review し、/review agent を使い、重要度ごとの checklist を作る</summary>

<img src="images/code-review-swimlane-single.png" alt="Code review workflow: review, identify issues, prioritize, generate checklist." width="800"/>

### Basic Review

この例では `@` symbol で file を参照し、その中身を Copilot CLI に直接読ませて review します。

```bash
copilot

> @samples/book-app-project/book_app.py の code quality をレビューして
```

---

<details>
<summary>🎬 動作例を見る</summary>

![Code Review Demo](images/code-review-demo.gif)

*Demo output varies. Your model, tools, and responses will differ from what's shown here.*

</details>

---

### Input Validation Review

review の観点を明示すると、Copilot CLI はその観点に集中して見てくれます。ここでは input validation に注目しています。

```text
copilot

> @samples/book-app-project/utils.py の input validation をレビューして。validation の不足、error handling の抜け、edge case を確認して
```

### Cross-File Project Review

`@` で directory 全体を参照すると、project 内の file をまとめて見てもらえます。

```bash
copilot

> @samples/book-app-project/ 全体をレビューして、見つかった issue を重要度別の markdown checklist にして
```

### Interactive Code Review

最初は broad に review して、そのあと follow-up question で深掘りしていきます。会話を切らずに続けられるのがポイントです。

```bash
copilot

> @samples/book-app-project/book_app.py をレビューして:
> - Input validation
> - Error handling
> - Code style と best practices

# Copilot CLI provides detailed review

> ユーザー入力の handling について、見落としている edge case はありますか？

# Copilot CLI shows potential issues with empty strings, special characters

> 見つかった issue を重要度順の checklist にして

# Copilot CLI generates prioritized action items
```

### Review Checklist Template

出力 format を指定して頼むと、issue に貼り付けやすい checklist を作ってもらえます。

```bash
copilot

> @samples/book-app-project/ をレビューして、見つかった issue を次の重要度別に markdown checklist にして:
> - Critical (data loss risks, crashes)
> - High (bugs, incorrect behavior)
> - Medium (performance, maintainability)
> - Low (style, minor improvements)
```

### Git Changes を理解する（/review の前に大事）

`/review` command を使う前に、git における 2 種類の変更を理解しておきましょう。

| Change Type | 意味 | 確認方法 |
|-------------|---------------|------------|
| **Staged changes** | `git add` で次の commit 対象としてマークした変更 | `git diff --staged` |
| **Unstaged changes** | 変更したが、まだ stage していない内容 | `git diff` |

```bash
# Quick reference
git status           # Shows both staged and unstaged
git add file.py      # Stage a file for commit
git diff             # Shows unstaged changes
git diff --staged    # Shows staged changes
```

### `/review` Command を使う

`/review` command は built-in の **code-review agent** を呼び出します。staged / unstaged changes を分析し、ノイズの少ない実用的な feedback を返してくれます。

```bash
copilot

> /review
# Invokes the code-review agent on staged/unstaged changes
# Provides focused, actionable feedback

> /review Check for security issues in authentication
# Run review with specific focus area
```

> 💡 **Tip**: code-review agent は pending changes があるときに特に力を発揮します。より焦点を絞った review が欲しいなら、先に `git add` しておくのがおすすめです。

</details>

---

<a id="workflow-2-refactoring"></a>
<details>
<summary><strong>Workflow 2: Refactoring</strong> - code を整理し、責務を分け、error handling を改善する</summary>

<img src="images/refactoring-swimlane-single.png" alt="Refactoring workflow: assess code, plan changes, implement, verify behavior." width="800"/>

### Simple Refactoring

> **最初に試すなら:** `@samples/book-app-project/book_app.py command handling で if/elif chains を使っています。dictionary dispatch pattern に refactor して`。

まずはシンプルな改善から始めましょう。各 prompt は `@` で file を参照しつつ、どんな refactor をしたいかを具体的に伝えています。

```bash
copilot

> @samples/book-app-project/book_app.py command handling で if/elif chains を使っています。dictionary dispatch pattern に refactor して

> @samples/book-app-project/utils.py のすべての function に type hints を追加して

> @samples/book-app-project/book_app.py にある本の表示ロジックを utils.py に切り出して、責務を分離して
```

> 💡 **refactoring に慣れていない場合**: まずは type hints の追加や変数名の改善のような小さい変更から始めると安心です。

---

<details>
<summary>🎬 動作例を見る</summary>

![Refactor Demo](images/refactor-demo.gif)

*Demo output varies. Your model, tools, and responses will differ from what's shown here.*

</details>

---

### 責務を分ける

1 つの prompt に複数 file を `@` で渡すと、Copilot CLI は file 間で code を移動するような refactor も提案できます。

```bash
copilot

> @samples/book-app-project/utils.py @samples/book-app-project/book_app.py
> utils.py では print statements と logic が混ざっています。display functions と data processing を分離するように refactor して
```

### Error Handling を改善する

関連する 2 つの file と concern を一緒に示すと、一貫した方針を考えてくれます。

```bash
copilot

> @samples/book-app-project/utils.py @samples/book-app-project/books.py
> これらの file では error handling が一貫していません。custom exceptions を使った統一方針を提案して
```

### Documentation を追加する

docstring に何を含めたいかを箇条書きで具体的に伝えると、出力の質が上がります。

```bash
copilot

> @samples/book-app-project/books.py のすべての method に詳しい docstring を追加して:
> - parameter の型と説明を含める
> - return value を記載する
> - 発生しうる exception を書く
> - usage example を追加する
```

### Tests を使った安全な Refactoring

先に test を作ってから refactor する流れです。既存挙動を守りながら安心して改善できます。

```bash
copilot

> @samples/book-app-project/books.py を refactoring する前に、現在の挙動に対する tests を生成して

# Get tests first

> 次に、file operations で context manager を使うように BookCollection class を refactor して

# Refactor with confidence - tests verify behavior is preserved
```

</details>

---

<a id="workflow-3-debugging"></a>
<details>
<summary><strong>Workflow 3: Debugging</strong> - bug を追い、security audit を行い、複数 file にまたがる issue をたどる</summary>

<img src="images/debugging-swimlane-single.png" alt="Debugging workflow: understand error, locate root cause, fix, test." width="800"/>

### Simple Debugging

> **最初に試すなら:** `@samples/book-app-buggy/books_buggy.py 「The Hobbit」が data にあるのに検索結果に出ないと報告されています。原因を debug して。`

まずは「何がおかしいか」を自然な言葉で説明するところから始めます。buggy な book app を使って、次のような debugging pattern を試してください。

```bash
copilot

# Pattern: "Expected X but got Y"
> @samples/book-app-buggy/books_buggy.py 「The Hobbit」が data にあるのに検索結果に出ないと報告されています。原因を debug して

# Pattern: "Unexpected behavior"
> @samples/book-app-buggy/book_app_buggy.py 存在しない本を削除しようとすると、app が削除できたと言います。なぜか調べて

# Pattern: "Wrong results"
> @samples/book-app-buggy/books_buggy.py 1冊だけ read にしたいのに、全部 read になってしまいます。bug は何ですか？
```

> 💡 **debugging のコツ**: *見えている症状* と *本来どうなるべきか* をセットで伝えると、Copilot CLI が原因にたどり着きやすくなります。

---

<details>
<summary>🎬 動作例を見る</summary>

![Fix Bug Demo](images/fix-bug-demo.gif)

*Demo output varies. Your model, tools, and responses will differ from what's shown here.*

</details>

---

### 「Bug Detective」- 関連 Bug も見つける

これが context-aware debugging の強みです。file 全体を `@` で渡し、ユーザー報告の symptom だけを説明してください。Copilot CLI は root cause をたどるだけでなく、近くにある別の issue にも気づくことがあります。

```bash
copilot

> @samples/book-app-buggy/books_buggy.py
>
> ユーザー報告: 「author 名の一部で検索しても本が見つからない」
> なぜこうなるのか debug して
```

**Copilot CLI がしてくれること:**
```
Root Cause: Line 80 uses exact match (==) instead of partial match (in).

Line 80: return [b for b in self.books if b.author == author]

The find_by_author function requires an exact match. Searching for "Tolkien"
won't find books by "J.R.R. Tolkien".

Fix: Change to case-insensitive partial match:
return [b for b in self.books if author.lower() in b.author.lower()]
```

**なぜ重要か**: Copilot CLI は file 全体を読み、bug report の context を理解したうえで、具体的な修正案まで示してくれます。

> 💡 **Bonus**: file 全体を分析するため、聞いていない *別の* issue に気づくこともあります。たとえば author search を見ている途中で、`find_book_by_title` の case-sensitivity bug まで見つけてくれるかもしれません。

### 現実の Security サイドバー

自分の code を debug することも大切ですが、production app の security vulnerability を理解することも重要です。次のように、知らない file に対して security issue の audit を依頼できます。

```bash
copilot

> @samples/buggy-code/python/user_service.py にある security vulnerability をすべて見つけて
```

この file には、production app でよく遭遇する real-world security pattern が含まれています。

> 💡 **出てきやすい security 用語:**
> - **SQL Injection**: user input をそのまま database query に埋め込み、攻撃者が悪意ある command を実行できてしまう状態
> - **Parameterized queries**: より安全な方法。placeholder (`?`) を使って user data と SQL command を分離する
> - **Race condition**: 2 つの処理が同時に進み、互いに干渉して想定外の結果になること
> - **XSS (Cross-Site Scripting)**: web page に悪意ある script を埋め込まれてしまう脆弱性

---

### Error を理解する

stack trace をそのまま prompt に貼り、あわせて `@` で関連 file を渡します。Copilot CLI が source code と結びつけて説明してくれます。

```bash
copilot

> 次の error が出ています:
> AttributeError: 'NoneType' object has no attribute 'title'
>     at show_books (book_app.py:19)
>
> @samples/book-app-project/book_app.py なぜ起きるのかと、どう直せばよいか説明して
```

### Test Case つきで Debug する

具体的な input と observed output を伝えると、再現しやすくなります。

```bash
copilot

> @samples/book-app-buggy/books_buggy.py remove_book function に bug があります。「Dune」を削除すると
> 「Dune Messiah」まで消えてしまいます。root cause を説明して修正案を出して
```

### コードの流れの中で Issue を追う

複数 file を参照して data flow をたどると、原因がどこで入り込んだかを追跡できます。

```bash
copilot

> ユーザーから、本の一覧番号が 1 ではなく 0 から始まると報告されています。
> @samples/book-app-buggy/book_app_buggy.py @samples/book-app-buggy/books_buggy.py
> list 表示の flow をたどって、どこで問題が起きているか特定して
```

### Data 問題を理解する

code だけでなく data file も一緒に見せることで、より現実的な error handling の提案が得られます。

```bash
copilot

> @samples/book-app-project/data.json @samples/book-app-project/books.py
> JSON file が壊れて app が落ちることがあります。これを graceful に扱うにはどうすればよいですか？
```

</details>

---

<a id="workflow-4-test-generation"></a>
<details>
<summary><strong>Workflow 4: Test Generation</strong> - 包括的な tests と edge cases を自動生成する</summary>

<img src="images/test-gen-swimlane-single.png" alt="Test Generation workflow: analyze function, generate tests, include edge cases, run." width="800"/>

> **最初に試すなら:** `@samples/book-app-project/books.py の全 function に対して、edge case を含む pytest tests を生成して`

### 「Test Explosion」- 2 個の test が 15+ 個に増える

手動で test を書くと、多くの開発者はまず次の 2〜3 個で止まりがちです。
- 正常系の test
- 異常系の test
- 1 つの edge case

Copilot CLI に包括的な test 生成を頼むとどうなるか見てみましょう。次の prompt は、`@` file reference と箇条書きを組み合わせて、丁寧な test coverage を促しています。

```bash
copilot

> @samples/book-app-project/books.py に対する包括的な pytest tests を生成して。次を含めて:
> - 本の追加
> - 本の削除
> - title での検索
> - author での検索
> - 既読化
> - 空 data に関する edge cases
```

---

<details>
<summary>🎬 動作例を見る</summary>

![Test Generation Demo](images/test-gen-demo.gif)

*Demo output varies. Your model, tools, and responses will differ from what's shown here.*

</details>

---

**得られるもの**: 15 個以上の tests が一気に出てきます。たとえば次のようなものです。

```python
class TestBookCollection:
    # Happy path
    def test_add_book_creates_new_book(self):
        ...
    def test_list_books_returns_all_books(self):
        ...

    # Find operations
    def test_find_book_by_title_case_insensitive(self):
        ...
    def test_find_book_by_title_returns_none_when_not_found(self):
        ...
    def test_find_by_author_partial_match(self):
        ...
    def test_find_by_author_case_insensitive(self):
        ...

    # Edge cases
    def test_add_book_with_empty_title(self):
        ...
    def test_remove_nonexistent_book(self):
        ...
    def test_mark_as_read_nonexistent_book(self):
        ...

    # Data persistence
    def test_save_books_persists_to_json(self):
        ...
    def test_load_books_handles_missing_file(self):
        ...
    def test_load_books_handles_corrupted_json(self):
        ...

    # Special characters
    def test_add_book_with_unicode_characters(self):
        ...
    def test_find_by_author_with_special_characters(self):
        ...
```

**結果**: 本来なら 1 時間かけて考えて書くような edge case test を、30 秒ほどで土台として用意できます。

---

### Unit Tests

1 つの function にしぼって、見てほしい input pattern を列挙すると、focused な unit test を作れます。

```bash
copilot

> @samples/book-app-project/utils.py の get_book_details に対する包括的な pytest tests を生成して。次をカバーして:
> - 正常な入力
> - 空文字
> - 不正な year format
> - とても長い title
> - author 名の special characters
```

### Tests を実行する

toolchain の command が分からないときも、自然な日本語で聞けば大丈夫です。

```bash
copilot

> tests を実行するにはどうすればいいですか？ pytest command を教えてください。

# Copilot CLI responds:
# cd samples/book-app-project && python -m pytest tests/
# Or for verbose output: python -m pytest tests/ -v
# To see print statements: python -m pytest tests/ -s
```

### 特定 Scenario の Test

少し難しい scenario を列挙すると、happy path だけでない test を出してくれます。

```bash
copilot

> @samples/book-app-project/books.py に対して、次の scenario の tests を生成して:
> - 重複した本の追加（同じ title と author）
> - title の部分一致で本を削除する
> - collection が空のときに本を探す
> - save 中の file permission error
> - book collection への同時 access
```

### 既存 File に追加 Test を足す

すでにある test file を前提に、「追加で」必要な cases を作ってもらうこともできます。

```bash
copilot

> @samples/book-app-project/books.py
> find_by_author function に対して、次の edge cases を含む追加 tests を生成して:
> - ハイフン入りの author 名（例: "Jean-Paul Sartre"）
> - 名が複数ある author
> - 空文字の author
> - アクセント付き文字を含む author 名
```

</details>

---

<a id="workflow-5-git-integration"></a>
<details>
<summary><strong>Workflow 5: Git Integration</strong> - commit messages、PR descriptions、/pr、/delegate、/diff</summary>

<img src="images/git-integration-swimlane-single.png" alt="Git Integration workflow: stage changes, generate message, commit, create PR." width="800"/>

> 💡 **この workflow では git の基本知識を前提としています**（staging、committing、branches）。git がまだ不慣れなら、先に他の 4 つから試しても大丈夫です。

### Commit Message を生成する

> **最初に試すなら:** `copilot -p "次の差分に対する conventional commit message を生成して: $(git diff --staged)"` — まず少し変更を stage してから、これを実行してみてください。

この例では、`-p` inline prompt flag と shell command substitution を使って、`git diff` の出力をそのまま Copilot CLI に渡しています。`$(...)` は中の command を実行し、その出力を外側の command に差し込みます。

```bash

# See what changed
git diff --staged

# Generate commit message using [Conventional Commit](../GLOSSARY.md#conventional-commit) format
# (structured messages like "feat(books): add search" or "fix(data): handle empty input")
copilot -p "次の差分に対する conventional commit message を生成して: $(git diff --staged)"

# Output: "feat(books): add partial author name search
#
# - Update find_by_author to support partial matches
# - Add case-insensitive comparison
# - Improve user experience when searching authors"
```

---

<details>
<summary>🎬 動作例を見る</summary>

![Git Integration Demo](images/git-integration-demo.gif)

*Demo output varies. Your model, tools, and responses will differ from what's shown here.*

</details>

---

### 変更内容を説明してもらう

`git show` の出力を `-p` prompt に流し込むと、直近 commit を分かりやすい言葉で要約してくれます。

```bash
# What did this commit change?
copilot -p "この commit が何をしているか説明して: $(git show HEAD --stat)"
```

### PR Description

`git log` と structured prompt を組み合わせて、pull request description を自動生成できます。

```bash
# Generate PR description from branch changes
copilot -p "次の変更に対する pull request description を生成して:
$(git log main..HEAD --oneline)

含める内容:
- 変更の要約
- 変更した理由
- 実施した testing
- Breaking changes の有無 (yes/no)"
```

### Interactive Mode で `/pr` を使う

current branch で作業しているなら、interactive mode の中で `/pr` command を使って pull request を扱えます。表示・作成・修正・自動判断などが可能です。

```bash
copilot

> /pr [view|create|fix|auto]
```

### Push 前の最終 Review

`git diff main..HEAD` を `-p` prompt に渡すと、push 前の sanity check として便利です。

```bash
# Last check before pushing
copilot -p "push 前に、これらの変更に issue がないかレビューして:
$(git diff main..HEAD)"
```

### `/delegate` で Background Task を任せる

`/delegate` command は、task を GitHub Copilot cloud agent に任せるための command です。明確に切り出せる作業を background で進めたいときに向いています。

```bash
copilot

> /delegate login form に input validation を追加して

# Or use the & prefix shortcut:
> & README header の typo を修正して

# Copilot CLI:
# 1. Commits your changes to a new branch
# 2. Opens a draft pull request
# 3. Works in the background on GitHub
# 4. Requests your review when done
```

ほかの作業をしながら並行して進めたい、よく定義された task に向いています。

### `/diff` で Session Changes を確認する

`/diff` command は、current session で行われた変更をまとめて見せてくれます。commit 前の見直しに役立ちます。

```bash
copilot

# After making some changes...
> /diff

# Shows a visual diff of all files modified in this session
# Great for reviewing before committing
```

</details>

---

## Quick Tip: Plan や Code の前に Research する

library の比較、best practice の確認、初めて触る topic の調査などには、先に `/research` を使うと判断がしやすくなります。

```bash
copilot

> /research CLI app の user input validation に向いている Python library は何ですか？
```

Copilot は GitHub repositories や web sources を調べ、reference 付きの summary を返してくれます。feature を作り始める前に方向性を固めたいときに便利です。結果は `/share` で共有することもできます。

> 💡 **Tip**: `/research` は `/plan` の前段にぴったりです。まず調べてから、次に実装 plan を立てましょう。

---

## Putting It All Together: Bug Fix Workflow

report された bug を修正するときの、ひと通りの流れは次のようになります。

```bash

# 1. Understand the bug report
copilot

> ユーザーから「author 名の一部では本を検索できない」と報告されています
> @samples/book-app-project/books.py 原因を分析して特定して

# 2. Debug the issue (continuing in same session)
> 分析結果を踏まえて、find_by_author function を見せて issue を説明して

> find_by_author function が部分一致に対応するよう修正して

# 3. Generate tests for the fix
> @samples/book-app-project/books.py 次の条件に対する pytest tests を生成して:
> - author 名の完全一致
> - author 名の部分一致
> - 大文字小文字を区別しない matching
> - author 名が見つからない場合

# 4. Generate commit message
copilot -p "次の差分に対する commit message を生成して: $(git diff --staged)"

# Output: "fix(books): support partial author name search"
```

### Bug Fix Workflow Summary

| Step | Action | Copilot Command |
|------|--------|-----------------|
| 1 | bug を理解する | `> [describe bug] @relevant-file.py Analyze the likely cause` |
| 2 | 詳細な分析を得る | `> 該当する function を見せて、issue を説明して` |
| 3 | 修正を実装する | `> その issue を修正して` |
| 4 | test を生成する | `> 必要な scenario に対する tests を生成して` |
| 5 | commit する | `copilot -p "次の差分に対する commit message を生成して: $(git diff --staged)"` |

---

# Practice

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

ここからは、これらの workflow を自分で使ってみましょう。

---

## ▶️ Try It Yourself

デモが終わったら、次の variation を試してみてください。

1. **Bug Detective Challenge**: `samples/book-app-buggy/books_buggy.py` の `mark_as_read` function を Copilot CLI に debug してもらいましょう。1 冊だけではなく ALL books が read になってしまう理由を説明できましたか？

2. **Test Challenge**: book app の `add_book` function の test を生成してみましょう。自分では思いつかなかった edge case がいくつ含まれているか数えてみてください。

3. **Commit Message Challenge**: book app file を何か 1 つ少し変更し、`git add .` したうえで次を実行します。
   ```bash
   copilot -p "次の差分に対する conventional commit message を生成して: $(git diff --staged)"
   ```
   自分で急いで書く message より、分かりやすくなりましたか？

**Self-Check**: 「debug this bug」が「find bugs」より強力な理由を説明できれば、この章の workflow の考え方をつかめています（context が効くからです）。

---

## 📝 Assignment

### Main Challenge: Refactor, Test, and Ship

ハンズオン例では `find_book_by_title` や code review を扱いました。今度は `book-app-project` の別の function で同じ workflow を練習してみましょう。

1. **Review**: `books.py` の `remove_book()` に edge cases や潜在的な issue がないか review してもらいます。
   `@samples/book-app-project/books.py remove_book() function をレビューして。title が別の本と部分一致した場合（例: "Dune" と "Dune Messiah"）はどうなりますか？ 未対応の edge case はありますか？`
2. **Refactor**: `remove_book()` を改善し、case-insensitive matching や、本が見つからないときの分かりやすい feedback を入れてもらいます
3. **Test**: 改善後の `remove_book()` 専用に pytest tests を生成します。対象は次のとおりです。
   - 既存の本を削除する
   - case-insensitive な title matching
   - 存在しない本に対して適切な feedback が返る
   - 空の collection から削除する
4. **Review**: 変更を stage して `/review` を実行し、残っている issue がないか確認します
5. **Commit**: 次の command で conventional commit message を生成します。
   `copilot -p "次の差分に対する conventional commit message を生成して: $(git diff --staged)"`

---

# Summary

## 🔑 Key Takeaways

| Skill | Chapter | できるようになったこと |
|-------|---------|----------------|
| Basic Commands | Ch 01 | interactive mode、plan mode、programmatic mode (`-p`)、slash commands を使える |
| Context | Ch 02 | `@` で file を参照し、session を管理し、context window を理解できる |
| Workflows | Ch 03 | code review、refactor、debug、test generation、git integration を実践できる |

Chapters 04-06 では、さらに強力な追加機能を学びます。

---

## 🛠️ 自分の Personal Workflow を育てる

GitHub Copilot CLI の使い方に「唯一の正解」はありません。自分のパターンを作っていくうえで、次の点が役立ちます。

> 📚 **Official Documentation**: 推奨 workflow や tips は [Copilot CLI best practices](https://docs.github.com/copilot/how-tos/copilot-cli/cli-best-practices) を参照してください。

- **non-trivial な task は `/plan` から始める**。良い plan は良い結果につながります
- **うまくいった prompt を残しておく**。失敗したときも原因をメモしておくと、自分だけの playbook になります
- **気軽に試行錯誤する**。長く詳細な prompt が合う人もいれば、短い prompt と follow-up を好む人もいます。自分に自然なやり方を見つけてください

> 💡 **次の章でやること**: Chapters 04 と 05 では、こうした best practice を custom instructions や skills として定着させる方法を学びます。

---

## ➡️ 次は？

残りの章では、Copilot CLI の能力をさらに広げる追加機能を扱います。

| Chapter | 内容 | こんなときに役立つ |
|---------|----------------|---------------------|
| Ch 04: Agents | specialized AI persona を作る | frontend や security の expert が欲しいとき |
| Ch 05: Skills | task ごとの instructions を auto-load する | 同じ prompt を何度も繰り返すとき |
| Ch 06: MCP | 外部 service と接続する | GitHub や database などの live data が欲しいとき |

**おすすめ**: まず core workflow を 1 週間ほど試してみて、その後必要を感じたタイミングで Chapters 04-06 に戻ってくるのが実践的です。

---

## 追加トピックへ進む

**[Chapter 04: Agents and Custom Instructions](../04-agents-custom-instructions/README.md)** では、次のことを学びます。

- built-in agents（`/plan`, `/review`）の使い方
- `.agent.md` files で専門 agent（frontend expert、security auditor など）を作る方法
- multi-agent collaboration patterns
- project standards を共有する custom instruction files

---

**[← Chapter 02 に戻る](../02-context-conversations/README.md)** | **[Chapter 04 へ進む →](../04-agents-custom-instructions/README.md)**
