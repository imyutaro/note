# Clean Architectureメモ

## 感想

- 断片的な知識を1つにまとめた感じで、自分としては勉強になった。
- 全体を通して行間を埋めて読むのが難しい気がした。
  - SOLID原則とか、アーキテクチャとか、Interfaceとか、ある程度わかっていないと本の中の説明だと足りない感じがした。
  - SOLID原則については、https://medium.com/carbon-consulting/solid-principles-with-python-245e45f9b1f8 を見るとPythonのサンプルコードもあって理解しやすいかもしれない。
  - 本に記載の課題を、どう具体的に実装して、なぜ課題が解決できるのかを理解するのが初学者には難しいかもしれない。なので、ある程度コード書ける人などに質問するなり、一緒に本を読むなりした方が初学者はいいかもしれない。
- あまり本筋に関係ない昔話があったり、日本語訳がおかしい部分があったりと、軽く読むだけにとどめたり、頭の中で変換したりしないといけなく、所々読みづらい部分があった。
  - 日本語意味わからないってなったら、原著読みにいった方がいい。https://github.com/GunterMueller/Books-3/blob/master/Clean%20Architecture%20A%20Craftsman%20Guide%20to%20Software%20Structure%20and%20Design.pdf
- https://nrslib.com/clean-architecture/ に記載されているYouTube動画 https://youtu.be/BvzjpAe3d4g が本を読み終わった後のサポートとして、とても良い。（ちなみに、この記事・動画の方は https://www.shoeisha.co.jp/book/detail/9784798151687 の著者の方。）

## メモ

p.133

> ソフトウェアを変更しづらくするには、多数のソフトウェアコンポーネントから依存されるようにすればいい。多数のコンポーネントから依存されたコンポーネントは非常に安定している。

この考え方はなかった。勉強になる。変更させたくないようなクラスや関数は、多数のソフトウェアから依存させれば、変更したらエラーとかいっぱい出て変更させるモチベーション減らせるって考え方だと思う。個人が変更させたくないからって依存増やしたらスパゲッティコードになるので、”変更させたくないようなクラスや関数”を正しく”チーム”で決定する必要はあるとは思う。

p.142

> 主系列からの距離

これは、何らかのツールで自動算出できそう。

p.165

> 優れたアーキテクチャがあれば、システムはモノリシックとして生まれ、単一ファイルでデプロイされ、独立してデプロイ可能な単位になるまで成長し、最後はサービスやマイクロサービスまでたどり着くことが可能になるだろう。その後、物事が変わったときには、これまでの流れを逆転させ、モノリシックまで戻せるようになっておくべきである。

モノリシック→マイクロサービスに変更できるように実装しておくべき。

p.172

簡単に実装するとこんな感じ？右と左どっちがいいのだろうか？ ~~個人的には、numpyかpandasかのソース読んだときに左みたいな実装見たことある気がするし、左の方がいい気がしている。~~

→ 基本は右（実装2）で実装して、抽象クラス使えないようなクラスを利用する場合は左（実装1）の実装をするような実装3がいい？

実装1

```python
class DatabaseInterface:
    def __init__(self):
        # - ここを変更するだけで、ビジネスロジック側はDBの変更が可能。
        # - もし、具象クラスのinterface（個々のreadとかstoreとかの関数名）が変わったとしても、
        #   DatabaseInterface内のstoreでの呼び出しを変えればいいだけ（具象クラスとのinterfaceの
        #   バインディングはDatabaseInterfaceが担っている）。
        #   - これは、サードパーティのクラスなどを利用する場合などに便利。
        #     （サードパーティのクラスは必ずしも自分たちの想定しているinterfaceではないことがあるため。）
        self.__db = Database1()

    def store(self):
        self.__db.store()

    def read(self):
        self.__db.read()

class Database1:
    def __init__(self):
        pass

    def store(self):
        print("store via db1.")

    def read(self):
        print("read via db1.")

class Database2:
    def __init__(self):
        pass

    def store(self):
        print("store via db2.")

    def read(self):
        print("read via db2.")

class BusinessRules:
    def __init__(self):
        pass

    def exec_process(self, db: DatabaseInterface):
        db.store()
        db.read()

def main():
    db: DatabaseInterface = DatabaseInterface()
    business_rules = BusinessRules()
    business_rules.exec_process(db)

main()
```

実装2

```python
from abc import ABCMeta, abstractmethod

class DatabaseInterface(metaclass=ABCMeta):
    @abstractmethod
    def store(self):
        raise NotImplementedError

    @abstractmethod
    def read(self):
        raise NotImplementedError

class Database1(DatabaseInterface):
    def __init__(self):
        pass

    def store(self):
        print("store via db1.")

    def read(self):
        print("read via db1.")

class Database2(DatabaseInterface):
    def __init__(self):
        pass

    def store(self):
        print("store via db2.")

    def read(self):
        print("read via db2.")

class BusinessRules:
    def __init__(self):
        pass

    def exec_process(self, db: DatabaseInterface):
        db.store()
        db.read()

def main():
    # - ここを変更するだけで、ビジネスロジック側はDBの変更が可能。
    #   - （`from db import Database1 as Database`とファイルの先頭でしておいて、
    #       import文の部分だけを変えればいいようにできる）
    # - 新たなDBは、Interfaceを必ず継承して都度実装する必要がある。
    # - なので、DatabaseのInterfaceは必ず最初に固定する必要がある。
    #   - Interfaceが変わる（read->loadに変わるなど）場合、`db.read()`と書かれている部分
    #     全てをdb.load()に書き換える必要があるため、Interfaceは変わらない方が好ましい。
    db: DatabaseInterface = Database1()
    business_rules = BusinessRules()

    if not isinstance(db, DatabaseInterface):
        raise TypeError(f"{type(db)} doesn't inherit DatabaseInterface.")
    business_rules.exec_process(db)

main()
```

実装3

```python
from abc import ABCMeta, abstractmethod

class DatabaseInterface(metaclass=ABCMeta):
    @abstractmethod
    def store(self):
        raise NotImplementedError

    @abstractmethod
    def read(self):
        raise NotImplementedError

class ThirdPartyDatabase:
    def __init__(self):
        pass

    def save(self):
        print("save via third party.")

    def load(self):
        print("load via third party.")

class Database3(DatabaseInterface):
    def __init__(self):
        self.__db = ThirdPartyDatabase()

    def store(self):
        self.__db.save()

    def read(self):
        self.__db.load()

class BusinessRules:
    def __init__(self):
        pass

    def exec_process(self, db: DatabaseInterface):
        db.store()
        db.read()

def main():
    db: DatabaseInterface = Database3()
    business_rules = BusinessRules()

    if not isinstance(db, DatabaseInterface):
        raise TypeError(f"{type(db)} doesn't inherit DatabaseInterface.")
    business_rules.exec_process(db)

main()
```


p.202

> また、このレイヤーには、外部サービスなどの外部の形式から、ユースケースやエンティティが使用する内部の形式にデータを変換するアダプターも含まれる。

データ形式を変える処理を行うレイヤー。DBなどに格納する形式が変更された場合は、このレイヤーのコードも影響を受ける？（ https://youtu.be/BvzjpAe3d4g?t=3619 らへんの話がここと同じ？）分析の前処理などのデータ変換もこのレイヤーに属すると思う。

p.207

> Humble（控えめ）

おそらく、ここの訳は建築のメタファーがあるので、「質素」が正しいと思う。Humble Objectはテストが難しいクラスなので、なるべく質素に（小さく）なっていた方がいいという意味だと思う。
https://bliki-ja.github.io/HumbleObject/

p.221

> 我々アーキテクトは、それがいつ必要になるかに気を配らなければいけない。また、境界を完全に構築しようとすると、コストが高くつくことを認識する必要がある。

境界を作るにはコストがかかる。なので、開発フェーズによって境界を作るかどうかを適切に判断しなければならない。

p.255

> 実装の詳細の可視性を制限しよう。実装の詳細は変更されると考えよう。コードが細部を知る場所を少なくしておけば、それだけコードを追跡・修正する場所も少なくなる。

可視性を制限し、実装の詳細を知られないようにすることで、実装の詳細に依存したコードが書かれるのを防ぐことができる。

p.288

> マイクロサービスやサービス指向アーキテクチャとの主な違いは、依存性を切り離す方法だ。モノリシックなアプリケーションのうまく作られたコンポーネントは、マイクロサービス化に向かう際の第一歩と見なすことができるだろう。

マイクロサービスやサービス指向アーキテクチャでは、サービスで分離することで依存性の切り離しを行うが、モノリシックではコンポーネントという形で依存性を切り離す。
