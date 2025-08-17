# TLA+ プロジェクト情報

このプロジェクトではTLA+ CLIツールが利用可能です。

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

## プロジェクト構造

- `/opt/tlaplus/` - TLA+ツール本体の配置場所
  - `tla2tools.jar` - コアツール
  - `CommunityModules-deps.jar` - 追加モジュール

## よく使うワークフロー

1. **仕様の作成と検証**
   ```bash
   # 1. TLA+仕様を作成
   # 2. 構文チェック
   sany MySpec.tla
   # 3. モデル検証
   tlc MySpec.tla
   ```

2. **PlusCalアルゴリズムの開発**
   ```bash
   # 1. PlusCalコードを書く
   # 2. TLA+に変換
   pcal Algorithm.tla
   # 3. 生成されたTLA+を検証
   tlc Algorithm.tla
   ```