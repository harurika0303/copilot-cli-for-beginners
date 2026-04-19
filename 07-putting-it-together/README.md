![第07章: Putting It All Together](images/chapter-header.png)

> **ここでは、これまで学んだことがすべてつながります。idea から merged PR までを 1 つの session で進めましょう。**

この章では、これまで学んだ内容を complete workflow としてまとめます。multi-agent collaboration で feature を作り、security issue を commit 前に検知する pre-commit hooks を設定し、Copilot を CI/CD pipeline に組み込み、feature idea から merged PR までを 1 つの terminal session で進める流れを見ていきます。ここで GitHub Copilot CLI は、本当の意味での force multiplier になります。

> 💡 **Note**: この章は「全部を組み合わせる」章です。ただし、productive になるために agents・skills・MCP が必須というわけではありません。基本 workflow — describe、plan、implement、test、review、ship — だけでも十分に強力です。

## 🎯 学習目標

この章を終えると、次のことができるようになります。

- agents、skills、MCP（Model Context Protocol）を unified workflow に組み込む
- 複数 tool を組み合わせて complete feature を仕上げる
- hooks で基本的な automation を導入する
- 実務的な best practice を workflow に反映する

> ⏱️ **想定時間**: 約75分（読む時間 15分 + ハンズオン 60分）

---

## 🧩 現実世界のたとえ: Orchestra

<img src="images/orchestra-analogy.png" alt="Orchestra Analogy - Unified Workflow" width="800"/>

オーケストラにはいくつものセクションがあります。
- **Strings** は土台を支える（core workflows のようなもの）
- **Brass** は力強さを加える（specialized agent のようなもの）
- **Woodwinds** は彩りを加える（skills のようなもの）
- **Percussion** はリズムを刻む（外部 system とつなぐ MCP のようなもの）

それぞれ単体でも役立ちますが、指揮がそろうと大きな力になります。

**この章で学ぶのは、その組み合わせ方です。**
*conductor が orchestra をまとめるように、agents、skills、MCP を 1 つの workflow にまとめていきます。*

まずは、code を変更し、tests を生成し、review し、PR を作るまでを 1 session で進める例から見ていきましょう。

---

## Idea から Merged PR までを 1 Session で

editor、terminal、test runner、GitHub UI を行き来して context を失うのではなく、1 つの terminal session でまとめて進めるのがこの章のテーマです。

```bash
# Start Copilot in interactive mode
copilot

> book app に、read が False の本だけを表示する "list unread" command を追加したいです。
> どの file を変更する必要がありますか？

# Copilot creates high-level plan...

# SWITCH TO PYTHON-REVIEWER AGENT
> /agent
# Select "python-reviewer"

> @samples/book-app-project/books.py get_unread_books method を設計して
> 最適な approach は何ですか？

# SWITCH TO PYTEST-HELPER AGENT
> /agent
# Select "pytest-helper"

> @samples/book-app-project/tests/test_books.py 未読本を絞り込むための
> test case を設計して

# IMPLEMENT
> books.py の BookCollection に get_unread_books method を追加して
> book_app.py に "list unread" command option を追加して
> show_help function の help text も更新して

# TEST
> この新 feature に対する包括的な tests を生成して

# Review the changes
> /review

# If review passes, use /pr to operate on the pull request for the current branch
> /pr [view|create|fix|auto]

# Or ask naturally if you want Copilot to draft it from the terminal
> "Feature: Add list unread books command" という title で pull request を作成して
```

**従来のやり方**: editor、terminal、test runner、docs、GitHub UI を行き来するたびに context が切れやすい。
**このやり方**: 1 つの session の中で流れを保ったまま進められます。

> 💡 **もっと大きな task では**: `/fleet` を使って、独立した subtasks を parallel に進めることもできます。

---

# Additional Workflows

<img src="images/combined-workflows.png" alt="People assembling a colorful giant jigsaw puzzle with gears, representing how agents, skills, and MCP combine into unified workflows" width="800"/>

Chapters 04-06 を終えた人向けに、agents、skills、MCP を組み合わせた workflow をいくつか紹介します。

## The Integration Pattern

組み合わせるときの mental model は次の 4 段階です。

<img src="images/integration-pattern.png" alt="The Integration Pattern - A 4-phase workflow: Gather Context (MCP), Analyze and Plan (Agents), Execute (Skills + Manual), Complete (MCP)" width="800"/>

---

## Workflow 1: Bug Investigation and Fix

実務でもそのまま使える bug fix workflow です。

```bash
copilot

# PHASE 1: Understand the bug from GitHub (MCP provides this)
> issue #1 の詳細を教えて

# PHASE 2: Research best practice (deep research with web + GitHub sources)
> /research Python で大文字小文字を区別しない文字列比較を行う best practice は？

# PHASE 3: Find related code
> @samples/book-app-project/books.py find_by_author method を見せて

# PHASE 4: Get expert analysis
> /agent
# Select "python-reviewer"

> この method に、部分一致の matching に関する issue がないか分析して

# PHASE 5: Fix with agent guidance
> lowercase comparison と 'in' operator を使って修正を実装して

# PHASE 6: Generate tests
> /agent
# Select "pytest-helper"

> find_by_author の部分一致に対する pytest tests を生成して
> test case には、部分一致、case variations、一致なし を含めて

# PHASE 7: Commit and PR
> この修正に対する commit message を生成して

> issue #1 に関連付けた pull request を作成して
```

---

## Workflow 2: Code Review Automation（optional）

> 💡 **この section は optional です。** pre-commit hook は team で便利ですが、最初から必須ではありません。
>
> ⚠️ **performance note**: この hook は staged file ごとに `copilot -p` を呼ぶため、commit が大きいと少し時間がかかります。

**git hook** は、git が特定のタイミングで自動実行する script です。たとえば commit 前に code review を走らせることができます。

```bash
# Create a pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

# Get staged files (Python files only)
STAGED=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.py$')

if [ -n "$STAGED" ]; then
  echo "Running Copilot review on staged files..."

  for file in $STAGED; do
    echo "Reviewing $file..."

    # Use timeout to prevent hanging (60 seconds per file)
    # --allow-all auto-approves file reads/writes so the hook can run unattended.
    # Only use this in automated scripts. In interactive sessions, let Copilot ask for permission.
    REVIEW=$(timeout 60 copilot --allow-all -p "Quick security review of @$file - critical issues only" 2>/dev/null)

    # Check if timeout occurred
    if [ $? -eq 124 ]; then
      echo "Warning: Review timed out for $file (skipping)"
      continue
    fi

    if echo "$REVIEW" | grep -qi "CRITICAL"; then
      echo "Critical issues found in $file:"
      echo "$REVIEW"
      exit 1
    fi
  done

  echo "Review passed"
fi
EOF

chmod +x .git/hooks/pre-commit
```

これで commit のたびに quick security review が走るようになります。

---

## Workflow 3: 新しい Codebase への Onboarding

新しい project に入ったときも、context、agents、MCP を使えば ramp-up がかなり速くなります。

```bash
# Start Copilot in interactive mode
copilot

# PHASE 1: Get the big picture with context
> @samples/book-app-project/ この codebase の高レベルな architecture を説明して

# PHASE 2: Understand a specific flow
> @samples/book-app-project/book_app.py ユーザーが
> "python book_app.py add" を実行したときの流れを説明して

# PHASE 3: Get expert analysis with an agent
> /agent
# Select "python-reviewer"

> @samples/book-app-project/books.py に design issue や
> error handling の不足、改善点があれば教えて

# PHASE 4: Find something to work on (MCP provides GitHub access)
> "good first issue" の label が付いた open issue を一覧して

# PHASE 5: Start contributing
> いちばん取り組みやすい open issue を選んで、修正 plan を作って
```

この workflow は、@ context、agents、MCP を 1 つの onboarding session にまとめた実践例です。

---

# Best Practices & Automation

作業をより効果的にする patterns と habits をまとめます。

---

## Best Practices

### 1. 分析の前に Context を集める

まず context を集めてから分析に入るのが基本です。

```bash
# Good
> issue #42 の詳細を教えて
> /agent
# Select python-reviewer
> この issue を分析して

# Less effective
> /agent
# Select python-reviewer
> login bug を直して
# Agent doesn't have issue context
```

### 2. Agents / Skills / Custom Instructions の違いを意識する

それぞれに得意な役割があります。

```bash
# Agents: Specialized personas you explicitly activate
> /agent
# Select python-reviewer
> この authentication code に security issue がないかレビューして

# Skills: Modular capabilities that auto-activate when your prompt
# matches the skill's description
> この code に対する包括的な tests を生成して

# Custom instructions (.github/copilot-instructions.md): Always-on
# guidance that applies to every session without switching or triggering
```

> 💡 **ポイント**: 違いは「何ができるか」だけでなく、「どう起動するか」にあります。agents は明示的、skills は自動、custom instructions は常時オンです。

### 3. Sessions を絞る

1 feature 1 session を意識すると整理しやすくなります。`/rename` と `/exit` を活用しましょう。

```bash
# Good: One feature per session
> /rename list-unread-feature
# Work on list unread
> /exit

copilot
> /rename export-csv-feature
# Work on CSV export
> /exit
```

### 4. Workflow を Repo に埋め込む

wiki に書くだけでなく、Copilot が読める形で repo に残すと効果が上がります。

- **Custom instructions** (`.github/copilot-instructions.md`): coding standard や build/test/deploy rules
- **Prompt files** (`.github/prompts/`): 再利用 가능한 prompt templates
- **Custom agents** (`.github/agents/`): specialist persona
- **Custom skills** (`.github/skills/`): auto-activate する workflow instructions

> 💡 **効果**: new team member も、repo を開くだけで team の workflow を使えるようになります。

---

## Bonus: Production Patterns

実務寄りの pattern もいくつか見ておきましょう。

### PR Description Generator

```bash
# Generate comprehensive PR descriptions
BRANCH=$(git branch --show-current)
COMMITS=$(git log main..$BRANCH --oneline)

copilot -p "次の内容に対する PR description を生成して:
Branch: $BRANCH
Commits:
$COMMITS

含める内容: Summary, Changes Made, Testing Done, Screenshots Needed"
```

### CI/CD Integration

既存の CI/CD pipeline に Copilot review を組み込むこともできます。詳しくは [CI/CD Integration](../appendices/ci-cd-integration.md) を参照してください。

---

# Practice

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

ここからは complete workflow を実際に試してみましょう。

---

## ▶️ Try It Yourself

1. **End-to-End Challenge**: 小さめの feature（例: unread books の一覧、CSV export など）を 1 つ選び、次の flow を試します。
   - `/plan` で plan を立てる
   - agents（python-reviewer、pytest-helper）で設計を固める
   - implement する
   - tests を作る
   - PR を作る

2. **Automation Challenge**: Code Review Automation workflow の pre-commit hook を設定し、意図的な vulnerability を含む file で commit を試してみてください。block されるか確認します。

3. **Your Production Workflow**: 自分がよく行う task を 1 つ選び、checklist にしてみましょう。どこを skills、agents、hooks で自動化できるか考えてみてください。

**Self-Check**: agents、skills、MCP がどう連携するか、そしてどんな場面でどれを使うべきかを同僚に説明できれば、この章の目標を達成できています。

---

## 📝 Assignment

### Main Challenge: End-to-End Feature

ハンズオン例では「list unread books」feature を扱いました。今度は別の feature、**year range で本を検索する機能** を作ってみましょう。

1. Copilot を起動し、context を読み込ませます: `@samples/book-app-project/books.py`
2. `/plan ユーザーが 2 つの年の間に出版された本を検索できる "search by year" command を追加したい` を実行します
3. `BookCollection` に `find_by_year_range(start_year, end_year)` method を追加します
4. `book_app.py` に `handle_search_year()` function を追加し、開始年と終了年を聞くようにします
5. tests を生成します: `@samples/book-app-project/books.py @samples/book-app-project/tests/test_books.py find_by_year_range() に対する tests を生成して。invalid years、reversed range、結果なし などの edge case を含めて`。
6. `/review` で変更を review します
7. README を更新します: `@samples/book-app-project/README.md 新しい "search by year" command の documentation を追加して`。
8. commit message を生成します

**成功条件**: idea → context → plan → implement → test → document → commit の流れを、Copilot CLI を使って自分で完了できることです。

> 💡 **Bonus**: Chapter 04 で custom agents を作っているなら、implementation review 用 agent や doc-writer agent も試してみてください。

<details>
<summary>💡 Hints（クリックして展開）</summary>

**上の ["Idea to Merged PR"](#idea-to-merged-pr-in-one-session) の流れをなぞれば OK です。**

考えるべき edge cases:
- `2000` と `1990` のように range が逆転している場合
- 該当する本が 1 冊もない場合
- numeric ではない input が入った場合

</details>

---

<details>
<summary>🔧 <strong>Common Mistakes</strong>（クリックして展開）</summary>

| Mistake | 起きること | Fix |
|---------|--------------|-----|
| いきなり implementation に入る | 後で直すのが高い design issue を見落とす | まず `/plan` を使う |
| 1 つの tool だけに頼る | 結果が遅い / 粗い | Agent → Skill → MCP の組み合わせを使う |
| commit 前に review しない | bug や security issue を見落とす | `/review` または pre-commit hook を使う |
| workflow を team と共有しない | 各人が毎回やり直す | agents、skills、instructions として repo に残す |

</details>

---

# Summary

## 🔑 Key Takeaways

1. **Integration > Isolation**: tools は組み合わせるほど効果が高い
2. **Context first**: 分析の前に必要な context を集める
3. **Agents、Skills、MCP の役割を使い分ける**
4. **繰り返しは automation する**: hooks や scripts が効果を増幅する
5. **workflow を document する**: team 全体の再利用性が上がる

> 📋 **Quick Reference**: コマンドやショートカットの一覧は [GitHub Copilot CLI command reference](https://docs.github.com/en/copilot/reference/cli-command-reference) を参照してください。

---

## 🎓 Course Complete!

お疲れさまでした。ここまでで次のことを学びました。

| Chapter | 学んだこと |
|---------|-------------------|
| 00 | Copilot CLI の installation と Quick Start |
| 01 | 3 つの interaction mode |
| 02 | `@` syntax を使った context management |
| 03 | development workflows |
| 04 | specialized agents |
| 05 | extensible skills |
| 06 | MCP による external connection |
| 07 | unified production workflow |

これで GitHub Copilot CLI を、開発 workflow の中で本格的な force multiplier として使う準備が整いました。

## ➡️ What's Next

学習はここで終わりではありません。次のような形で活用を広げていけます。

1. **daily practice**: 実際の作業で Copilot CLI を使う
2. **custom tools を作る**: 自分の project に合う agents や skills を作る
3. **knowledge を共有する**: team に workflow を広める
4. **最新情報を追う**: GitHub Copilot の新機能を定期的に確認する

### Resources

- [GitHub Copilot CLI Documentation](https://docs.github.com/copilot/concepts/agents/about-copilot-cli)
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)
- [Community Skills](https://github.com/topics/copilot-skill)

---

**ここまで来たら、ぜひ実際の project で何か 1 つ作ってみてください。**

**[← Chapter 06 に戻る](../06-mcp-servers/README.md)** | **[コース Home に戻る →](../README.md)**
