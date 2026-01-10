# TLA+ プロジェクト情報

このプロジェクトでは TLA+ CLI ツールが利用可能です。

## 利用可能なコマンド

### tlc - TLA+ Model Checker

```bash
# 基本的な使い方
tlc MySpec.tla

# 設定ファイルを指定
tlc -config MySpec.cfg MySpec.tla

# ワーカー数を指定（並列実行）
tlc -workers 4 MySpec.tla

# デッドロック検出を無効化
tlc -deadlock MySpec.tla
```

### pcal - PlusCal Translator

```bash
# PlusCalコードをTLA+に変換
pcal MyAlgorithm.tla

# Cスタイルの構文を使用
pcal -c MyAlgorithm.tla
```

### sany - TLA+ Syntax Analyzer

```bash
# 構文チェック
sany MySpec.tla
```

### tla2tex - TLA+ to LaTeX Converter

```bash
# TLA+仕様をLaTeX形式に変換
tla2tex MySpec.tla

# メタディレクトリを指定
tla2tex -metadir /path/to/metadir MySpec.tla
```

### apalache-mc - Apalache Model Checker

TLC とは異なる SMT ベースのモデルチェッカー。シンボリック実行により、大きな状態空間でも効率的に検証可能。

```bash
# 基本的なモデル検査
apalache-mc check MySpec.tla

# 不変条件を指定して検査
apalache-mc check --inv=Invariant MySpec.tla

# 複数の不変条件を検査
apalache-mc check --inv=Inv1,Inv2 MySpec.tla

# CONSTANTSの初期化演算子を指定
apalache-mc check --cinit=ConstInit --inv=Invariant MySpec.tla

# 探索の深さを指定（デフォルト: 10）
apalache-mc check --length=20 MySpec.tla

# 時相論理プロパティを検査
apalache-mc check --temporal=Property MySpec.tla

# デッドロック検出を無効化
apalache-mc check --no-deadlock MySpec.tla

# 型チェックのみ実行
apalache-mc typecheck MySpec.tla

# 構文解析のみ実行
apalache-mc parse MySpec.tla

# シミュレーションモード
apalache-mc simulate --max-run=100 MySpec.tla
```

#### Apalache 用の型アノテーション

Apalache では型アノテーションが必要です：

```tla
CONSTANTS
    \* @type: Int;
    N,
    \* @type: Seq(Int);
    INPUT_SEQ

VARIABLES
    \* @type: Int;
    counter,
    \* @type: Bool;
    flag
```

#### 出力ディレクトリ

- 結果は `./_apalache-out/` に保存される
- 反例は `violation1.tla` として出力される

## よく使うワークフロー

1. **仕様の作成と検証（Apalache）**

   ```bash
   # 1. TLA+仕様を作成（型アノテーション付き）
   # 2. 型チェック
   apalache-mc typecheck MySpec.tla
   # 3. 不変条件の検証
   apalache-mc check --inv=Invariant MySpec.tla
   ```

2. **PlusCal アルゴリズムの開発**

   ```bash
   # 1. PlusCalコードを書く（型アノテーション付き）
   # 2. TLA+に変換
   pcal Algorithm.tla
   # 3. 型チェック
   apalache-mc typecheck Algorithm.tla
   # 4. 生成されたTLA+を検証
   apalache-mc check --inv=Invariant Algorithm.tla
   ```

3. **シンボリック実行による検証**

   ```bash
   # CONSTANTSを動的に生成して検証
   apalache-mc check --cinit=ConstInit --inv=Invariant MCMySpec.tla
   ```
