# 良いコードを書く技術

- 著者: 縣俊貴（あがたとしたか）
- 出版社: 技術評論者
- 版: 第2版（2021/5/11）
- ISBN: 978-4-297-12048-1 C3055
- URL: -
- 読み始め: 2026-07-10
---

## p.95　オブジェクトを参照することが多い処理は、そのオブジェクト自身にメソッドを持たせる（疑問）
### 日付、ステータス
- 日付: 2026-07-12
- status: 未回収
### コード例
1. 改善前
```typescript
type Order = {
  items: { price: number; quantity: number }[];
  couponRate: number; // 例: 0.1 = 10% 引き
};

function calculateTotal(order: Order): number {
  const subtotal = order.items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );

  return subtotal * (1 - order.couponRate);
}
```
2. 改善後
```typescript
class Order {
  constructor(
    private readonly items: { price: number; quantity: number }[],
    private readonly couponRate: number
  ) {}

  total(): number {
    const subtotal = this.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );

    return subtotal * (1 - this.couponRate);
  }
}

const order = new Order(
  [{ price: 1000, quantity: 2 }],
  0.1
);

console.log(order.total()); // 1800
```
3. 呼び出し側
```typescript
// 改善前
calculateTotal(order);

// 改善後
order.total();
```
4. 解説（改善後のメリット）
- 呼び出し側が注文の内部構造を知らなくてよくなる。
- 将来、割引ルールや税計算が変わっても、`Order` の中を変更すれば済みます。
- ただし、何でもメソッドに移すわけではありません。たとえば「複数の注文を横断して集計する」「外部APIへ送信する」といった、**単一のオブジェクトだけに属しにくい処理は、別のサービスや関数に置くことが多い**です。
---
## p.97 特定の値が重複する場合は、定数にまとめ繰り返し使用できるようにする（わかったこと）
### 日付、ステータス
- 日付: 2026-07-12
- status: 未回収
### コード例
1. 改善前
```typescript
function calculateShippingFee(totalPrice: number): number {
  if (totalPrice >= 5000) {
    return 0;
  }

  return 500;
}

function showFreeShippingMessage(totalPrice: number): string {
  if (totalPrice >= 5000) {
    return "送料無料です";
  }

  return "あと5000円以上で送料無料です";
}
```
2. 改善後
```typescript
const FREE_SHIPPING_THRESHOLD = 5000;
const SHIPPING_FEE = 500;

function calculateShippingFee(totalPrice: number): number {
  if (totalPrice >= FREE_SHIPPING_THRESHOLD) {
    return 0;
  }

  return SHIPPING_FEE;
}

function showFreeShippingMessage(totalPrice: number): string {
  if (totalPrice >= FREE_SHIPPING_THRESHOLD) {
    return "送料無料です";
  }

  return `あと${FREE_SHIPPING_THRESHOLD}円以上で送料無料です`;
}
```
3. 解説
- 改善前のコードでは、変更漏れが起きやすくなる。
- 改善前のコードでは、5000` が複数回登場している。送料無料の条件を 6,000 円に変えるとき、すべて修正しなければならない。
- 改善後のコードでは、定数名が値の意味も説明してくれる（マジックナンバーの解消）。
---
## p.99 列挙型`enum`にまとめる
### 日付、ステータス
- 日付: 2026-07-12
- status: 未回収
### コード例
0. `enum`にまとめるとは？
- 「取り得る値が決まっているもの」を名前付きの選択肢として定義すること。
1. 改善前
```typescript
type Order = {
  status: string;
};
function canCancel(order: Order): boolean {
  return order.status === "pending";
}
const order: Order = {
  status: "pending",
};
```
2. 改善後
```typescript
enum OrderStatus {
  Pending = "pending",
  Shipped = "shipped",
  Delivered = "delivered",
  Cancelled = "cancelled",
}
type Order = {
  status: OrderStatus;
};
function canCancel(order: Order): boolean {
  return order.status === OrderStatus.Pending;
}
const order: Order = {
  status: OrderStatus.Pending,
};
// order.status = "peding"; // エラーになる
order.status = OrderStatus.Shipped;
```
3. 解説
値の候補を一か所で管理できる。
- `OrderStatus` を見れば、どんな状態があるか分かります。
- タイプミスを防げる  
  `"peding"` のような存在しない状態を代入すると、TypeScript がエラーにします。
- 変更しやすい  
  `"pending"` という内部表現を変更する必要があっても、`enum` の定義を直せば利用箇所への影響を抑えられます。
---
## p.XX 一行要約
### 日付、ステータス
- 日付: YYYY-MM-DD
- status: 未回収

---
## p.XX 一行要約
### 日付、ステータス
- 日付: YYYY-MM-DD
- status: 未回収

---
