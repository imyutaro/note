# SQL Coding Style

SQLのコーディングスタイルについて調べた。あんまり、記事もなかったので、有名なスタイルとかないのかな？そこまで決めなくても、みんな同じような書き方になるから？

UDFs (User-Defined Functions)のことを考えると、全て小文字だとUDFかどうかの区別がつきづらいので、予約語・関数は大文字の方がいいのかなぁ。

## SQL Style Guide

- [SQL Style Guide](https://www.sqlstyle.guide/)
- [SQLスタイルガイド・SQL style guide by Simon Holywell](https://www.sqlstyle.guide/ja/)
- [treffynnon/sqlstyle.guide - GitHub](https://github.com/treffynnon/sqlstyle.guide)

感想とまとめたものを書く。

- 個人的な感想
  - 基本的な命名規則などは良いが、クエリのフォーマットが個人的に好きでない。
  - 理由としては以下。
    - 取得するカラムが同じ行にあることで、ぱっと見でいくつカラムがあるかわからない
    - > SELECT、FROM、その他キーワードはすべて右揃えされ、列名や実装の詳細は左揃えは書く時に、めんどくさい。フォーマッターなどを使って整形を後から行う方が現実的。
    - カッコがどこにかかっているか、わかりづらい。

エイリアス等の変数の命名規則は[上記のドキュメント](https://www.sqlstyle.guide/ja/#%E5%91%BD%E5%90%8D%E8%A6%8F%E5%89%87)参照。
命名規則の項目で言われている、ハンガリアン記法がなぜ非推奨なのかは下記を参照。

- [ハンガリアン記法 メモ](https://qiita.com/inabe49/items/a4eedae3b7c44fe04c0d)
- [Why shouldn't I use "Hungarian Notation" ?](https://stackoverflow.com/a/111972)

大事そうな部分だけ書く。（と思ってたらほとんど書いてしまった...）

### 命名規則

#### 表

- 集合名詞を使用するか、望ましくはないが、複数形を使用する。例えば、employeesよりstaffが望ましい。
- tblなどの接頭辞やハンガリアン記法による接頭辞を付けない。
- 表に列の名前と同じ名前を付けない。その逆も同様。
- 関連付けする表の名前に2つの表名を連結した名前を付けないようにする。cars_mechanicsよりservicesが望ましい。

#### 列

- 常に単数形の名前を使用する。
- 主キーに単純にidを使うのは極力避ける。
- 表と同じ名前の列を追加しない。逆もまた同様。
- 固有名詞など意味のある場合を除いて、常に小文字を使用する。

#### エイリアス

- 何らかの方法でオブジェクトまたはその別名へ関連付けをすべきである。
- 大雑把な方法としてエイリアスはオブジェクト名の先頭文字を使う。
- すでに同じエイリアスがある場合は数値を追加する。
- 常にASキーワードを記載する。

```sql
SELECT first_name AS fn
  FROM staff AS s1
  JOIN students AS s2
    ON s2.mentor_id = s1.staff_num;
```

#### 統一的接尾辞

- _id - 主キーである列など一意の識別子。
- _status - flag値または任意のタイプの状況を表す（例: publication_status）。
- _total - 値の集合の合計または総計。
- _num - フィールドに数値が含まれていることを表す。
- _name - first_nameのように名前を強調する。
- _seq - 連続した数値を含む。
- _date - 何かの日付を含む列であることを表す。
- _tally - カウント。
- _size - ファイルサイズや衣類などのサイズ。
- _addr - 有形無形のデータのアドレス（例: ip_addr)。

### クエリの書き方

#### 予約語

`SELECT`や`WHERE`などの予約語は常に大文字にする。

```sql
SELECT
  user_id
FROM
  item_master
;
```

#### 空白

- `SELECT`など、予約語が全て同じ位置で終わるよに整列させる。
- `SELECT`や`FROM`などの予約語は全て右揃えし、列名などは左揃え。
- 以下の場合は、スペースを入れる
  - イコール(=)の前後、カンマ(,)、アポストロフィ(')の外側（括弧内またはカンマやセミコロンが後に来る場合は除く）

```sql
(SELECT f.species_name,
        AVG(f.height) AS average_height, AVG(f.diameter) AS average_diameter
   FROM flora AS f
  WHERE f.species_name = 'Banksia'
     OR f.species_name = 'Sheoak'
     OR f.species_name = 'Wattle'
  GROUP BY f.species_name, f.observation_date)

  UNION ALL

(SELECT b.species_name,
        AVG(b.height) AS average_height, AVG(b.diameter) AS average_diameter
   FROM botanic_garden_flora AS b
  WHERE b.species_name = 'Banksia'
     OR b.species_name = 'Sheoak'
     OR b.species_name = 'Wattle'
  GROUP BY b.species_name, b.observation_date);
```

#### 行間

- 以下の場合は改行を入れる
  - `AND`と`OR`の前
  - クエリ間を分けるために、セミコロン(;)の後
  - キーワードの定義後（よくわからん）
  - 複数のカラムをグループに分けたカンマの後
  - コードを関連するセクションに分ける時（よくわからん）

```sql
INSERT INTO albums (title, release_date, recording_date)
VALUES ('Charcoal Lane', '1990-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000'),
       ('The New Danger', '2008-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000');

UPDATE albums
   SET release_date = '1990-01-01 01:01:01.00000'
 WHERE title = 'The New Danger';

SELECT a.title,
       a.release_date, a.recording_date, a.production_date -- grouped dates together
  FROM albums AS a
 WHERE a.title = 'Charcoal Lane'
    OR a.title = 'The New Danger';
```

#### インデント

##### Join

Joinする際は、必要に応じて改行でグループ化する。

```sql
SELECT r.last_name
  FROM riders AS r
       INNER JOIN bikes AS b
       ON r.bike_vin_num = b.vin_num
          AND b.engine_tally > 2

       INNER JOIN crew AS c
       ON r.crew_chief_last_name = c.last_name
          AND c.chief = 'Y';
```

##### SubQueries

サブクエリでも同様に、今まで説明した規則に従い書くが、インデントは下げる。

```sql
SELECT r.last_name,
       (SELECT MAX(YEAR(championship_date))
          FROM champions AS c
         WHERE c.last_name = r.last_name
           AND c.confirmed = 'Y') AS last_championship_year
  FROM riders AS r
 WHERE r.last_name IN
       (SELECT c.last_name
          FROM champions AS c
         WHERE YEAR(championship_date) > '2008'
           AND c.confirmed = 'Y');
```

## SQLスタイルガイド - Qiita

- [SQLスタイルガイド - Qiita](https://qiita.com/taise/items/18c14d9b01a5dfd6d35e)

SQL Style Guideのめんどうな部分がなくなっている感じ。

- 個人的な感想
  - 基本的にはSQL Style Guideと同様。違いは以下。
    - めんどくさすぎる予約語右揃えでなく、左揃え
    - カラム毎に改行を入れる
  - ~~大文字、小文字に関しては、Cookpadの記事の主張に同意なので、個人的には予約語を大文字で書きたくない。（Shift押さなくちゃいけなくてしんどいし。）~~ UDFのこと考えたら、まぁ、いいかなっていう

## Cookpad

- [分析SQLのコーディングスタイル - クックパッド開発者ブログ](https://techlife.cookpad.com/entry/2016/11/09/000033)

感想とまとめたものを書く。

- 個人的な感想
  - 賛同できる主張。だが予約語を大文字で書きたい勢が多くいる気がする、、、
  - 以下の記述には、完全に同意。SQL Style Guideがそうだけど、めんどくさすぎる。
    流石にめんどくさいからフォーマッターがあるのかと思ったけど、ないっぽいし...どうやってるんだ...？
    > 4. 各句のキーワードはインデントしない
    > 4つめの話題。ちょっとあまりに面倒くさすぎて信じられないのですが、
    > 世の中には次のようにキーワードを右寄せにしたがる派閥も存在しています

1. 全て小文字
2. 改行前後のカンマと演算子は次の行の先頭に置く
3. 文末のセミコロンは単独行に置く
4. 各句のキーワードはインデントしない
   ```sql
   select count(*)
     from mst.users
    where user_id = 5;
   ```
   ↑こういうことをしない
5. join式はfromよりもインデントする
   ```sql
   from
       user_master as u
       inner join source.score as s on u.user_id = s.user_id
       inner join source.score_comp as c using (attr_id)
   ```
6. インデントはスペース4つ（with句についても言及されているが開発環境に依存することなので割愛）

この記事に則ったSQLの一例。

```sql
select
    user_id
    , user_session_id
    , row_number() over (
          partition by user_id, user_session_id
          order by log_time
      ) as session_step
    , log_time
    , keywords
from (
    select
        user_id
        , user_session_id
        , log_time
        , trim(word1
              ||' '|| coalesce(word2,'')
              ||' '|| coalesce(word3,'')
              ||' '|| coalesce(word4,'')
              ||' '|| coalesce(word5,'')
              ||' '|| coalesce(word6,'')
          ) as keywords
    from
        user_master
)
where
    not user_id is null
    and updated_at is null
;
```

## Future 株式会社 - SQLコーディング規約（Oracle）


- [SQLコーディング規約（Oracle） | Future Enterprise Coding Standards](https://future-architect.github.io/coding-standards/documents/forSQL/SQL%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E8%A6%8F%E7%B4%84%EF%BC%88Oracle%EF%BC%89.html#%E5%89%8D%E6%8F%90%E6%9D%A1%E4%BB%B6)

感想だけ書く。

- 個人的な感想
  - パフォーマンスについても言及されていて、結構良い感じ
  - 内容もいい感じ

## Other Links

- [標準SQL徹底入門](http://www.sql-post.biz/)

