# 複数フォルダ内の全ファイルを数字順にまとめて出力するPythonスクリプト

## はじめに

プログラミング学習のレッスンファイルなど、複数のフォルダに分かれたファイルを一覧化したいことってありますよね。この記事では、フォルダ名を**数字順（lesson1, lesson2, ..., lesson10, lesson11）**に正しくソートして、全ファイルの内容を1つのテキストファイルにまとめるPythonスクリプトを紹介します。

## 実現したいこと

- 複数フォルダ（lesson1～lesson13など）内の全ファイルを収集
- フォルダ名を数字順に正しくソート（lesson10がlesson2の後に来ないように）
- フォルダ構造とファイル内容を見やすく整形して出力
- 文字コードエラーにも対応

## 完成コード

```python
import os
from pathlib import Path
import re

def natural_sort_key(path):
    """数字を考慮した自然順ソート用のキー"""
    parts = re.split(r'(\d+)', str(path))
    return [int(part) if part.isdigit() else part.lower() for part in parts]

def collect_all_files(directory):
    output_file = "全ファイルまとめ.txt"
    base_path = Path(directory)
    
    with open(output_file, 'w', encoding='utf-8') as out:
        out.write("=" * 80 + "\n")
        out.write("ディレクトリ構造\n")
        out.write("=" * 80 + "\n\n")
        
        # フォルダを数字順にソート
        folders = sorted([f for f in base_path.iterdir() if f.is_dir()], key=natural_sort_key)
        
        for folder in folders:
            out.write(f"📁 {folder.name}/\n")
            files = sorted([f for f in folder.rglob('*') if f.is_file()], key=natural_sort_key)
            for file in files:
                relative = file.relative_to(folder)
                indent = '  ' * (len(relative.parts) - 1)
                out.write(f'  {indent}📄 {relative}\n')
            out.write('\n')
        
        out.write("\n")
        out.write("=" * 80 + "\n")
        out.write("ファイル内容\n")
        out.write("=" * 80 + "\n\n")
        
        # フォルダごとに数字順に処理
        for folder in folders:
            out.write(f"\n{'#'*80}\n")
            out.write(f"### {folder.name} ###\n")
            out.write(f"{'#'*80}\n\n")
            
            files = sorted([f for f in folder.rglob('*') if f.is_file()], key=natural_sort_key)
            
            for file_path in files:
                relative_path = file_path.relative_to(base_path)
                
                out.write(f"\n{'='*80}\n")
                out.write(f"📄 ファイル: {relative_path}\n")
                out.write(f"{'='*80}\n\n")
                
                try:
                    with open(file_path, 'r', encoding='utf-8') as f:
                        content = f.read()
                        out.write(content)
                        out.write("\n\n")
                except UnicodeDecodeError:
                    try:
                        with open(file_path, 'r', encoding='shift-jis') as f:
                            content = f.read()
                            out.write(content)
                            out.write("\n\n")
                    except Exception as e:
                        out.write(f"[読み込みエラー: {e}]\n\n")
                except Exception as e:
                    out.write(f"[読み込みエラー: {e}]\n\n")
        
        # 統計情報
        total_files = sum(1 for _ in base_path.rglob('*') if _.is_file())
        total_folders = len(folders)
        
        out.write(f"\n{'='*80}\n")
        out.write(f"統計: フォルダ数 {total_folders}個、ファイル数 {total_files}個\n")
        out.write(f"{'='*80}\n")
    
    print(f"✅ 完了! {output_file} に出力しました")
    print(f"📊 フォルダ数: {total_folders}個、ファイル数: {total_files}個")

# 実行例
if __name__ == "__main__":
    collect_all_files(r"C:\Users\username\Desktop\python_programming")
```

## 使い方

1. 上記コードを `collect_files.py` として保存
2. スクリプト内の `directory` パスを対象フォルダに変更
3. コマンドプロンプトまたはターミナルで実行

```bash
python collect_files.py
```

4. `全ファイルまとめ.txt` が生成される

## コードの解説

### 自然順ソート関数

```python
def natural_sort_key(path):
    """数字を考慮した自然順ソート用のキー"""
    parts = re.split(r'(\d+)', str(path))
    return [int(part) if part.isdigit() else part.lower() for part in parts]
```

通常の文字列ソートでは `lesson10` が `lesson2` より前に来てしまいます。この関数は文字列を数字部分と文字部分に分割し、数字部分は整数として扱うことで正しい順序を実現します。

**ソート結果の違い:**
- 通常のソート: `lesson1, lesson10, lesson11, lesson2, lesson3...`
- 自然順ソート: `lesson1, lesson2, lesson3, ..., lesson10, lesson11...`

### 文字コードエラー対応

```python
try:
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()
except UnicodeDecodeError:
    try:
        with open(file_path, 'r', encoding='shift-jis') as f:
            content = f.read()
    except Exception as e:
        out.write(f"[読み込みエラー: {e}]\n\n")
```

まずUTF-8で読み込みを試み、失敗した場合はShift-JISでリトライします。これにより、日本語環境で作成された古いファイルにも対応できます。

## 出力例

```
================================================================================
ディレクトリ構造
================================================================================

📁 lesson1/
  📄 main.py
  📄 test.py

📁 lesson2/
  📄 exercise.py

...

================================================================================
ファイル内容
================================================================================

################################################################################
### lesson1 ###
################################################################################

================================================================================
📄 ファイル: lesson1\main.py
================================================================================

print("Hello World")

...

================================================================================
統計: フォルダ数 13個、ファイル数 45個
================================================================================
```

## カスタマイズ例

### 特定の拡張子のみ収集

```python
# .pyファイルのみ収集
files = sorted([f for f in folder.rglob('*.py') if f.is_file()], key=natural_sort_key)
```

### 出力ファイル名を変更

```python
output_file = f"まとめ_{base_path.name}.txt"  # フォルダ名を含める
```

### バイナリファイルを除外

```python
# テキストファイルのみ処理
text_extensions = ['.py', '.txt', '.md', '.json', '.csv']
if file_path.suffix in text_extensions:
    # 処理を実行
```

## まとめ

このスクリプトを使えば、複数フォルダに分散したファイルを簡単に一覧化できます。学習ログの整理やコードレビュー時の資料作成に便利です。

### ポイント
- ✅ 自然順ソートで数字を正しく扱う
- ✅ フォルダ構造を視覚的に表示
- ✅ 文字コードエラーに対応
- ✅ 統計情報で全体像を把握

ぜひ活用してみてください！

## 参考
- [Pathlib公式ドキュメント](https://docs.python.org/ja/3/library/pathlib.html)
- [正規表現 re モジュール](https://docs.python.org/ja/3/library/re.html)