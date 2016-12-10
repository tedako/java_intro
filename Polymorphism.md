# ポリモーフィズム（Polymorphism）
## 代入とキャスト（18.1節）
- オブジェクトでは、継承関係にある型同士は、同じ種類の型と看做される。
  - スーパークラス型への代入(pp.419-)
    - 代入可能。
    - **オブジェクトは代入によって変化しない。**
    - スーパークラス型変数では、サブクラスで拡張したメンバにはアクセスできない。（スーパークラスからアクセス出来ないだけで、保持していることに注意）
  - サブクラス型への代入(pp.421-)
    - コンパイルエラー。
  - キャストによる強制的な代入とinstanceof演算子(pp.422-)
    - 強制的に「**スーパークラスのオブジェクトを、サブクラスへキャスト（ダウンキャスト）**」することでコンパイルエラーを避けることができる。ただし、スーパークラスが知らない拡張部分に対する処理を放置したままオブジェクトを扱うことになるため、危険。原則として避けるべき。
    - ただし、元々オブジェクトがスーパークラス型であるにも関わらずサブクラスにキャストされてただけならば（拡張部分に対するメンバを保持してるオブジェクトならば）、ダウンキャストしても良い。
      - ``変数名 instanceof 型名``
        - instanceof演算子は、指定した型名やそのサブクラスに該当する場合でもtrueを返す。
        - コード例: pp.424

<hr>

## オーバーライド（18.2節）
- オーバーライド概要(pp.426)
  - 継承したメソッドの機能を変更（≒上書き）すること。
    - OverloadとOverrideの違い: [コード例:ExamplePolymorphism](https://github.com/naltoma/ExamplePolymorphism)
    - どういう時に変更するのか？
      - 継承しただけでは役に立たない状態のメソッドや、変更しなければコンパイルエラーになるメソッドがある場合。（大まかな仕組みのみをスーパークラスで作り、機能詳細はサブクラスに一任するケース）
        - e.g., [java.util.AbstractList](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractList.html) は、「public abstract class AbstractList<E>」として宣言されている。abstractなclassは、実装していないメソッドがあることを示しており、継承先で実装する必要がある。
          - 未実装メソッドの例：「abstract E	get(int index)」
    - どう実現するのか？
      - 同じ名前のメソッドを継承先で作る。その際には「メソッドのアクセス修飾子、戻り値型、引数の型・数・並び等」は原則として同じである必要がある。（これらが異なると、オーバーロード(p.276)になる）
        - ただし、戻り値型がオブジェクトの場合は、サブクラス型に変えても良い。
        - アクセス修飾子は、より広範囲の広いものにだけ変更しても良い。
          - 上記の未実装 AbstractList.get() についてオーバーライドするなら、継承先で「E get(int index)」メソッドを実装する必要がある。
- コード例(pp.426-)
  - [Object.toString()](http://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#toString--) をオーバーライドしてみる。
  - toString()メソッドがあると、System.out.println()のような出力メソッドが自動で呼び出してくれる。
  - アノテーション
    - @Override
      - オーバーライドであることをアノテーションで書いておくと、誤ってオーバーライドになっていない場合（e.g., 引数の順序が間違ってた）にコンパイルエラーにして教えてくれる。
- オーバーライドメソッドを持つオブジェクトの構造(p.431)

<hr>

## ポリモーフィズム（18.3節）
- ポリモーフィズムとは
  - 教科書による説明
    - 同じ型の変数でも、どんなオブジェクトが入っているかで動作が変わってしまうこと。
    - 同じ型の変数でも、違う型のオブジェクトを入れることで様々な機能に変身させられるということ。
  - 別の説明例
    - オブジェクトを大まかに扱うことで、プログラマが楽をする。
      - 例えば、道路交通シミュレーションすることを想像してみよう。
        - 道路を利用する（道路上を移動する）対象は、車（バス、ダンプカー、中型トラック、自家用車）、バイク（自動二輪、原付き）、自転車、歩行者（一般人、子供、不注意者）、、、
          - これらを「道路上を移動するオブジェクト Vehicle」というスーパークラスを用意した上で、各々継承したサブクラスとして実装する。スーパークラスには「1秒間動く move()メソッド」を用意しておき、具体的な実装は各サブクラスで用意する。
            - [クラス図の例](./figs/Polymorphism.svg.xml.svg)
        - シミュレータのmainメソッドで、各クラスのインスタンスを用意する。
          - 用意した全オブジェクトに対して「1秒間動く move()メソッド」を実行させる際に、ポリモーフィズムがない状況だと「クラス毎にmove()メソッドを呼び出す」必要がある。これは、クラス数が多いと大変だし、無駄でもある。
            - ポリモーフィズムがある状況だと、全クラスのスーパークラスが持つメソッド Vehicle.move() を実行するだけで良い。

```
# 擬似コード1（ポリモーフィズムがない状況）
Bus[] buses = new Bus[3];
Bus[0] = new Bus();
Bus[1] = new Bus();
Bus[2] = new Bus();
DumpCar[] dumpCars = new DumpCar[2];
DumpCar[0] = new DumpCar();
DumpCar[1] = new DumpCar();

for(Bus bus: buses){
  bus.move();
}
for(DumpCar dumpCar: dumpCars){
  dumpCar.move();
}
// 以下、サブクラス毎に同じfor文が続く。


# 擬似コード2（ポリモーフィズムがある状況）
Vehicle[] vehicles = new Vehicle[5];
vehicles[0] = new Bus();
vehicles[1] = new Bus();
vehicles[2] = new Bus();
vehicles[3] = new DumpCar();
vehicles[4] = new DumpCar();

// どれだけサブクラスが増えたとしても、
// シミュレータ側では下記ループ文だけで全オブジェクトを操作できる。
for(Vehicle vehicle: vehicles){
  vehicle.move();
  //オーバーライドにより、オブジェクト自身のメソッドが実行される。
}
```

- ポリモーフィズム
  - どう実現するのか？
    - スーパークラスへの代入とオーバーライド
      - 最初からうまくクラス設計できる訳でもないので、まずは継承を意識してクラス設計してみよう。その後の実装を通して、「まとめて処理したい」「同一視したいクラス」等が出てきた際に、設計を見直そう。
  - コード例
    - ポリモーフィズムの利用、汎用システムの作成(pp.437-)
      - 多言語挨拶システム