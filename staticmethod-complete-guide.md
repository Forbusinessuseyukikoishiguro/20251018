# 🐰 デコレーターとメソッドの種類 完全ガイド

**ふわふわ大福店のうさうさ店長で学ぶ、ゼロからわかる実践的解説**

---

## 📚 目次

1. [そもそもデコレーターとは](#1-そもそもデコレーターとは)
2. [デコレーターの仕組み](#2-デコレーターの仕組み)
3. [3つのメソッド完全比較](#3-3つのメソッド完全比較)
4. [インスタンスメソッド詳細](#4-インスタンスメソッド詳細)
5. [クラスメソッド詳細](#5-クラスメソッド詳細)
6. [スタティックメソッド詳細](#6-スタティックメソッド詳細)
7. [使い分け判断フローチャート](#7-使い分け判断フローチャート)

---

## 1. そもそもデコレーターとは

### 📖 超シンプルな定義

**デコレーター = 関数やメソッドに「魔法」をかける記号**

`@` マークで始まる、関数の直前に書く特別な記法。

### 🎯 デコレーターの例え話

```
たい焼き屋での「ラッピング」

🥮 たい焼き（本体）
  ↓
📦 箱に入れる（デコレーター）
  ↓
🎀 リボンをつける（別のデコレーター）
  ↓
✨ 完成品（装飾された関数）

たい焼き自体は変わらないが、
見た目や機能が追加される！
```

### 💻 デコレーターの基本構文

```python
# デコレーターなし（普通の関数）
def hello():
    print("こんにちは")

hello()  # こんにちは

# ==========================================

# デコレーターあり
@some_decorator  # ← これがデコレーター
def hello():
    print("こんにちは")

# some_decorator が hello に「魔法」をかける
```

### 🔍 Pythonでよく使うデコレーター

| デコレーター | 何をする？ | 使う場面 |
|------------|----------|---------|
| `@property` | メソッドを属性のように使える | `obj.price` で呼べる |
| `@classmethod` | クラスメソッドに変換 | クラス全体の処理 |
| `@staticmethod` | スタティックメソッドに変換 | 補助関数 |
| `@abstractmethod` | 抽象メソッド（実装を強制） | 継承先で必ず実装 |
| `@dataclass` | データクラスを自動生成 | 簡単なクラス定義 |

---

## 2. デコレーターの仕組み

### 📖 デコレーターの正体

デコレーターは「関数を受け取って、新しい関数を返す関数」

### 💻 デコレーターの内部動作

```python
# ==========================================
# デコレーターの仕組みを理解する
# ==========================================

def my_decorator(func):
    """
    デコレーター本体
    
    引数:
        func: デコレートされる関数
    
    返り値:
        wrapper: 新しい関数
    """
    def wrapper(*args, **kwargs):
        """ラッパー関数（本体を包む）"""
        print("🎀 前処理: 関数の実行前")
        
        # 元の関数を実行
        result = func(*args, **kwargs)
        
        print("🎀 後処理: 関数の実行後")
        return result
    
    return wrapper


# ==========================================
# 使い方1: @を使う（推奨）
# ==========================================
@my_decorator
def greet(name):
    """挨拶する関数"""
    print(f"こんにちは、{name}さん！")

greet("うさうさ")

# 出力:
# 🎀 前処理: 関数の実行前
# こんにちは、うさうささん！
# 🎀 後処理: 関数の実行後


# ==========================================
# 使い方2: 手動で適用（@の正体）
# ==========================================
def greet2(name):
    print(f"こんにちは、{name}さん！")

# これは↑の @my_decorator と同じ意味
greet2 = my_decorator(greet2)

greet2("もちもち")
# 同じ出力が得られる


# ==========================================
# @記号は構文糖衣（シンタックスシュガー）
# ==========================================
# @my_decorator
# def func():
#     pass
#
# ↓ これと同じ意味
#
# def func():
#     pass
# func = my_decorator(func)
```

### 🎯 デコレーターの実用例

```python
# ==========================================
# 例1: 実行時間を測定するデコレーター
# ==========================================
import time

def measure_time(func):
    """実行時間を測定"""
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"⏱️ 実行時間: {end - start:.4f}秒")
        return result
    return wrapper

@measure_time
def slow_function():
    """重い処理"""
    time.sleep(1)
    print("処理完了")

slow_function()
# 出力:
# 処理完了
# ⏱️ 実行時間: 1.0012秒


# ==========================================
# 例2: ログを記録するデコレーター
# ==========================================
def log_function(func):
    """関数の呼び出しをログに記録"""
    def wrapper(*args, **kwargs):
        print(f"📝 関数 {func.__name__} を呼び出し")
        print(f"   引数: {args}, {kwargs}")
        result = func(*args, **kwargs)
        print(f"   結果: {result}")
        return result
    return wrapper

@log_function
def add(a, b):
    return a + b

add(5, 3)
# 出力:
# 📝 関数 add を呼び出し
#    引数: (5, 3), {}
#    結果: 8


# ==========================================
# 例3: 認証チェックをするデコレーター
# ==========================================
def require_auth(func):
    """認証が必要な処理"""
    def wrapper(self, *args, **kwargs):
        if not hasattr(self, 'is_authenticated') or not self.is_authenticated:
            print("❌ 認証されていません")
            return None
        return func(self, *args, **kwargs)
    return wrapper

class Shop:
    def __init__(self):
        self.is_authenticated = False
    
    @require_auth
    def sell(self, quantity):
        """認証が必要な販売処理"""
        print(f"✅ {quantity}個販売しました")

shop = Shop()
shop.sell(5)  # ❌ 認証されていません

shop.is_authenticated = True
shop.sell(5)  # ✅ 5個販売しました
```

### ⚠️ デコレーターの注意点

```python
# 注意1: デコレーターの順番は重要
@decorator1
@decorator2
def func():
    pass

# これは以下と同じ:
# func = decorator1(decorator2(func))
# 下から順番に適用される！


# 注意2: 複数のデコレーターを重ねられる
@log_function
@measure_time
def process():
    time.sleep(0.5)
    print("処理中")

# まずmeasure_timeが適用され、次にlog_functionが適用される


# 注意3: functools.wraps を使うべき
from functools import wraps

def my_decorator(func):
    @wraps(func)  # ← これを付けないと、元の関数の情報が失われる
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

---

## 3. 3つのメソッド完全比較

### 📊 一覧比較表

| 項目 | インスタンスメソッド | クラスメソッド | スタティックメソッド |
|------|-------------------|--------------|-------------------|
| **デコレーター** | なし | `@classmethod` | `@staticmethod` |
| **第一引数** | `self`（自分） | `cls`（クラス） | なし（自由） |
| **呼び出し方** | `obj.method()` | `Class.method()` | `Class.method()` |
| **インスタンス変数** | ✅ 使える | ❌ 使えない | ❌ 使えない |
| **クラス変数** | ✅ 使える | ✅ 使える | ❌ 使えない |
| **インスタンス必要？** | ✅ 必要 | ❌ 不要 | ❌ 不要 |
| **用途** | 個別の処理 | クラス全体の処理 | 補助関数 |
| **例** | 在庫を減らす | 統計を取る | 税額を計算 |

### 💻 3つのメソッドを一度に見る

```python
class DaifukuShop:
    """
    大福店クラス
    3つのメソッドの違いを理解する
    """
    
    # ========================================
    # クラス変数（全店舗共通）
    # ========================================
    total_shops = 0    # 総店舗数
    base_price = 150   # 基本価格
    tax_rate = 0.10    # 消費税率
    
    def __init__(self, owner_name, stock):
        """コンストラクタ"""
        # ========================================
        # インスタンス変数（各店舗個別）
        # ========================================
        self.owner_name = owner_name  # 店長名
        self.stock = stock            # 在庫数
        self.sold = 0                 # 販売数
        
        # クラス変数を更新
        DaifukuShop.total_shops += 1
    
    # ========================================
    # 1️⃣ インスタンスメソッド（デコレーターなし）
    # ========================================
    def sell(self, quantity):
        """
        販売処理（個別の店舗の処理）
        
        特徴:
        - デコレーターなし
        - 第一引数は self（必須）
        - self.xxx でインスタンス変数にアクセス
        - この店舗の在庫を減らす
        """
        # インスタンス変数を使う
        if quantity > self.stock:
            print(f"❌ {self.owner_name}店長: 在庫不足！")
            return False
        
        # インスタンス変数を変更
        self.stock -= quantity
        self.sold += quantity
        
        # クラス変数も使える
        price = quantity * DaifukuShop.base_price
        
        print(f"💰 {self.owner_name}店長: {quantity}個販売（¥{price}）")
        return True
    
    def show_status(self):
        """この店舗の状態を表示"""
        print(f"\n{'='*40}")
        print(f"🏪 {self.owner_name}店長の店")
        print(f"📦 在庫: {self.stock}個")
        print(f"💰 累計販売: {self.sold}個")
        print(f"{'='*40}\n")
    
    # ========================================
    # 2️⃣ クラスメソッド（@classmethod）
    # ========================================
    @classmethod
    def get_stats(cls):
        """
        全店舗の統計（クラス全体の処理）
        
        特徴:
        - @classmethod デコレーター必須
        - 第一引数は cls（クラス自身）
        - cls.xxx でクラス変数にアクセス
        - インスタンスを作らずに呼べる
        """
        # クラス変数にアクセス
        return f"📊 総店舗数: {cls.total_shops}店"
    
    @classmethod
    def from_dict(cls, data):
        """
        辞書からインスタンスを生成（ファクトリーメソッド）
        
        特徴:
        - 代替コンストラクタ
        - cls() でインスタンスを作成
        - 別の方法でオブジェクトを作れる
        """
        # cls() はクラス名() と同じ
        return cls(
            owner_name=data["owner"],
            stock=data.get("stock", 10)
        )
    
    @classmethod
    def change_price(cls, new_price):
        """
        基本価格を変更（全店舗に影響）
        
        特徴:
        - クラス変数を変更
        - すべてのインスタンスに影響する
        """
        cls.base_price = new_price
        print(f"📢 基本価格を¥{new_price}に変更しました")
    
    # ========================================
    # 3️⃣ スタティックメソッド（@staticmethod）
    # ========================================
    @staticmethod
    def calculate_tax(price):
        """
        税込価格を計算（補助関数）
        
        特徴:
        - @staticmethod デコレーター必須
        - 第一引数なし（self も cls も不要）
        - インスタンス変数もクラス変数も使わない
        - 純粋な計算処理
        - インスタンスを作らずに呼べる
        """
        # 引数だけを使う
        return int(price * 1.1)
    
    @staticmethod
    def validate_stock(stock):
        """
        在庫数のバリデーション（検証）
        
        特徴:
        - インスタンスに依存しない検証
        - インスタンス作成前にチェックできる
        """
        return isinstance(stock, int) and stock >= 0
    
    @staticmethod
    def format_price(price):
        """
        価格をフォーマット（文字列整形）
        
        特徴:
        - 単純な変換処理
        - ユーティリティ関数
        """
        return f"¥{price:,}"


# ==========================================
# 使い方の違いを実演
# ==========================================

print("="*60)
print("3つのメソッドの使い方の違い")
print("="*60)

# ========================================
# 1️⃣ インスタンスメソッド: インスタンスから呼ぶ
# ========================================
print("\n【1️⃣ インスタンスメソッド】")
shop = DaifukuShop("うさうさ", 20)
shop.sell(5)         # インスタンスから呼ぶ
shop.show_status()   # この店舗の情報

# ========================================
# 2️⃣ クラスメソッド: クラスから呼べる
# ========================================
print("\n【2️⃣ クラスメソッド】")

# インスタンス不要で呼べる
print(DaifukuShop.get_stats())

# ファクトリーメソッド
data = {"owner": "もちもち", "stock": 15}
shop2 = DaifukuShop.from_dict(data)
print(DaifukuShop.get_stats())  # 店舗数が増えた

# 全店舗に影響
DaifukuShop.change_price(180)

# ========================================
# 3️⃣ スタティックメソッド: クラスから呼べる
# ========================================
print("\n【3️⃣ スタティックメソッド】")

# インスタンス不要で呼べる
price = 1000
tax_price = DaifukuShop.calculate_tax(price)
print(f"税込価格: {DaifukuShop.format_price(tax_price)}")

# バリデーション（インスタンス作成前）
is_valid = DaifukuShop.validate_stock(20)
print(f"在庫数20は有効？ {is_valid}")

is_valid = DaifukuShop.validate_stock(-5)
print(f"在庫数-5は有効？ {is_valid}")
```

### 🎯 動作の違いを図解

```
┌────────────────────────────────────────┐
│ DaifukuShop クラス                      │
├────────────────────────────────────────┤
│ クラス変数:                             │
│  - total_shops = 2                     │
│  - base_price = 150                    │
│  - tax_rate = 0.10                     │
├────────────────────────────────────────┤
│ メソッド:                               │
│                                         │
│ インスタンスメソッド sell(self, qty)    │
│  ├─ self.stock にアクセス              │
│  └─ インスタンスが必要 ✅               │
│                                         │
│ @classmethod get_stats(cls)            │
│  ├─ cls.total_shops にアクセス         │
│  └─ インスタンス不要 ❌                 │
│                                         │
│ @staticmethod calculate_tax(price)     │
│  ├─ 引数だけ使う                        │
│  └─ インスタンス不要 ❌                 │
└────────────────────────────────────────┘
         │                    │
         ↓                    ↓
  ┌──────────┐        ┌──────────┐
  │ shop1    │        │ shop2    │
  ├──────────┤        ├──────────┤
  │ owner:   │        │ owner:   │
  │ うさうさ  │        │ もちもち  │
  │ stock: 15│        │ stock: 12│
  └──────────┘        └──────────┘
```

---

## 4. インスタンスメソッド詳細

### 📖 定義

**インスタンスメソッド = 各オブジェクト（インスタンス）の処理**

### 🎯 特徴5つ

1. **デコレーター不要**（何も付けない）
2. **第一引数は self**（必須）
3. **インスタンス変数にアクセス可能**
4. **クラス変数にもアクセス可能**
5. **最も一般的なメソッド**

### 💻 詳細な例

```python
class DaifukuShop:
    """インスタンスメソッドの詳細"""
    
    # クラス変数
    base_price = 150
    
    def __init__(self, owner_name, stock):
        # インスタンス変数
        self.owner_name = owner_name
        self.stock = stock
        self.sold = 0
        self.revenue = 0
    
    # ========================================
    # インスタンスメソッド
    # ========================================
    
    def sell(self, quantity):
        """
        販売処理
        
        selfで何ができる？
        - self.xxx でインスタンス変数にアクセス
        - self.method() で他のメソッドを呼ぶ
        - ClassName.xxx でクラス変数にアクセス
        """
        # 1. インスタンス変数を参照
        if quantity > self.stock:
            return False
        
        # 2. インスタンス変数を変更
        self.stock -= quantity
        self.sold += quantity
        
        # 3. クラス変数を参照
        price = quantity * DaifukuShop.base_price
        
        # 4. インスタンス変数を更新
        self.revenue += price
        
        # 5. 他のインスタンスメソッドを呼ぶ
        self._log_sale(quantity, price)
        
        return True
    
    def _log_sale(self, quantity, price):
        """
        内部メソッド（protected）
        
        _（アンダースコア）で始まる = 内部用
        """
        print(f"📝 {self.owner_name}店長: {quantity}個販売（¥{price}）")
    
    def get_profit_rate(self):
        """
        利益率を計算
        
        インスタンスメソッドは:
        - この店舗のデータを使う
        - 計算結果を返す
        """
        if self.revenue == 0:
            return 0.0
        
        cost = self.sold * 80  # 原価80円と仮定
        profit = self.revenue - cost
        return (profit / self.revenue) * 100
    
    def transfer_stock(self, other_shop, quantity):
        """
        他の店舗に在庫を譲渡
        
        インスタンスメソッドは:
        - 自分（self）のデータを操作
        - 他のインスタンス（other_shop）も操作できる
        """
        if quantity > self.stock:
            print("❌ 在庫不足")
            return False
        
        # 自分の在庫を減らす
        self.stock -= quantity
        
        # 相手の在庫を増やす
        other_shop.stock += quantity
        
        print(f"📦 {self.owner_name} → {other_shop.owner_name}: {quantity}個譲渡")
        return True


# 使用例
shop1 = DaifukuShop("うさうさ", 20)
shop2 = DaifukuShop("もちもち", 10)

shop1.sell(5)
print(f"利益率: {shop1.get_profit_rate():.1f}%")

shop1.transfer_stock(shop2, 5)
print(f"うさうさ店: {shop1.stock}個")
print(f"もちもち店: {shop2.stock}個")
```

### ⚠️ インスタンスメソッドの注意点

```python
class Shop:
    def sell(quantity):  # ❌ selfを忘れている
        pass

shop = Shop()
# shop.sell(5)  # エラー！

# 正しくは:
class Shop:
    def sell(self, quantity):  # ✅ selfが必要
        pass
```

---

## 5. クラスメソッド詳細

### 📖 定義

**クラスメソッド = クラス全体に関わる処理**

### 🎯 特徴5つ

1. **@classmethod デコレーター必須**
2. **第一引数は cls**（クラス自身）
3. **クラス変数にアクセス可能**
4. **インスタンス不要で呼べる**
5. **ファクトリーメソッドに最適**

### 💻 詳細な例

```python
class DaifukuShop:
    """クラスメソッドの詳細"""
    
    # クラス変数
    total_shops = 0
    total_revenue = 0
    company_name = "ふわふわ大福株式会社"
    
    def __init__(self, owner_name, stock):
        self.owner_name = owner_name
        self.stock = stock
        
        # クラス変数を更新
        DaifukuShop.total_shops += 1
    
    # ========================================
    # クラスメソッド
    # ========================================
    
    @classmethod
    def get_company_info(cls):
        """
        会社情報を取得
        
        clsで何ができる？
        - cls.xxx でクラス変数にアクセス
        - cls() でインスタンスを作成
        - cls.method() で他のクラスメソッドを呼ぶ
        """
        return f"{cls.company_name}（店舗数: {cls.total_shops}店）"
    
    @classmethod
    def from_dict(cls, data):
        """
        辞書からインスタンスを作成（ファクトリーメソッド）
        
        用途:
        - JSONデータからオブジェクト生成
        - 別の形式からの変換
        - 代替コンストラクタ
        """
        return cls(
            owner_name=data["owner"],
            stock=data.get("stock", 10)
        )
    
    @classmethod
    def from_csv(cls, csv_line):
        """
        CSV行からインスタンスを作成
        
        クラスメソッドは:
        - 様々な形式からインスタンスを作れる
        - __init__とは違う作り方を提供
        """
        parts = csv_line.split(',')
        return cls(
            owner_name=parts[0].strip(),
            stock=int(parts[1].strip())
        )
    
    @classmethod
    def create_franchise(cls, owner_name):
        """
        フランチャイズ店舗を作成
        
        クラスメソッドは:
        - 標準的な設定でインスタンスを作る
        - ビジネスロジックを含む
        """
        print(f"🏪 {owner_name}様、フランチャイズ契約ありがとうございます！")
        return cls(owner_name, stock=20)  # 標準在庫20個
    
    @classmethod
    def get_total_revenue(cls):
        """
        全店舗の総売上を取得
        
        クラスメソッドは:
        - クラス全体の統計を取る
        - すべてのインスタンスに関わる情報
        """
        return f"総売上: ¥{cls.total_revenue:,}"
    
    @classmethod
    def reset_stats(cls):
        """
        統計をリセット
        
        クラスメソッドは:
        - クラス変数を変更できる
        - 全インスタンスに影響する
        """
        cls.total_shops = 0
        cls.total_revenue = 0
        print("📊 統計をリセットしました")


# ========================================
# 使用例
# ========================================

# インスタンスを作らずに呼べる
print(DaifukuShop.get_company_info())

# ファクトリーメソッド（様々な形式から作成）
data = {"owner": "うさうさ", "stock": 25}
shop1 = DaifukuShop.from_dict(data)

csv = "もちもち,15"
shop2 = DaifukuShop.from_csv(csv)

shop3 = DaifukuShop.create_franchise("ぴょんぴょん")

# 統計情報
print(DaifukuShop.get_company_info())
```

### ⚠️ クラスメソッドの注意点

```python
class Shop:
    # @classmethod を忘れている
    def get_total(cls):  # ❌ デコレーターなし
        return cls.total

# Shop.get_total()  # エラー！

# 正しくは:
class Shop:
    @classmethod  # ✅ デコレーター必須
    def get_total(cls):
        return cls.total
```

---

## 6. スタティックメソッド詳細

### 📖 定義

**スタティックメソッド = クラスに属する補助関数**

### 🎯 特徴5つ

1. **@staticmethod デコレーター必須**
2. **第一引数なし**（self も cls も不要）
3. **インスタンス変数もクラス変数も使わない**
4. **純粋な計算・変換処理**
5. **名前空間の整理に便利**

### 💻 詳細な例

```python
class DaifukuShop:
    """スタティックメ