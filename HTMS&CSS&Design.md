# 1冊で身につくHTML & CSSとWebデザイン入門講座

知らなかったことを書いていく。

## 1章
1章は特に新しい学びはなかったと思う。

## 2章 HTML
- aタグでは、href="mailto:ogtsn99@gmail.com"のようにすることでメールをリンクとして貼ることができる
- ulタグとliタグで箇条書きができる。ulはunordered listといういう意味。
- ulをolに変えると数字付きの箇条書きになる。1. 2. 3. みたいな

tableタグでテーブルを作ることができる。border=1とすると線を引いてくれる。

thを利用してテーブルの項目部分、tdを使ってデータ部分を入力できる。

colspan="2"のようにすることで、二列分の大きさにできる。縦に繋げるのであればrowspan

formタグでフォームを作ることができる。属性にはaction（リクエスト送信先のURL）, method（POSTかGETか）, name（フォームの名前）を指定できる。

フォームで使うパーツとして最初に挙げられるのが、inputタグである。以下のinputでは基本的なテキスト入力ボックスが作成される。
また、placeholderタグで、デフォルトで表示されている文字列を指定できる。
```html
<form action="example.php" method="get" name="example">
    <input type="text" placeholder="placeholder here">
</form>
```

属性値にemailやurlを指定すると**フォームの送信前に自動的に検証される**。便利。

radioを指定してチェックボタン形式のフォームを作れる。ここでポイントはnameは同じにすること、valueはそれぞれ異なる文字列、そしてinputの後に表示したい値を書くこと。
```html
    <input type="radio" name="gender" value="男" checked> 男
    <input type="radio" name="gender" value="女"> 女
    <input type="radio" name="gender" value="どちらでもない"> どちらでもない
```

type="submit"で提出される。また、"image"を指定するとsubmitと同様の機能を持った上でボタンを画像に変える事ができる

selectとoptionタグで選択形式のフォームを作れる。multipleを選ぶと複数選べる（あまり見ないけど）。
```html
<select name="bloodtype" multiple>
    <option value="O" selected>O</option>
    <option value="A">A</option>
    <option value="AB">AB</option>
    <option value="B">B</option>
</select>
```

textareaは複数行にわたるテキストを入力できる。

labelを使うことでそのパーツ全体がクリック可能になる。radioやcheckboxで利用した場合、丸ではなく文字をクリックしてもチェックできて便利。o
labelはforでidを指定すれば良いので簡単。
```html
    <input type="checkbox" name="travel" value="ヨーロッパ" id="europe">
    <label for="europe">ヨーロッパ</label>
    <input type="checkbox" name="travel" value="アジア" id="asia">
    <label for="asia">アジア</label>
    <input type="submit" value="送信する">
```
識別名は必ずファイル内で重複させてはならない。

### グループ化

sectionタグで内容を分ける事ができる。そのままでは表示に何も変わりはないが、CSSで色付けしたりすることで見た目を変える事ができる。

header要素とhead要素とは異なるので注意。headerはページ上部の要素を囲むものである。

ナビゲーションメニューはnavタグによって作成できる。headerと一緒に使うのが良さそう。

読み物、記事の部分ではarticleが利用できる。

ページのメインコンテンツにはmainタグ、補足情報はasideタグを使う。

フッター部分にはfooterタグ。

どの用途にも当てはまらないものにはdivタグを使う。

## 3章 CSS

CSSはCascading Style Sheets

CSSの適用方法の一つ目はcssファイルを書く事である。headないにlinkタグを使ってhref属性にcssファイルを指定すれば良い。基本はこれを使う。
```html
<link rel="stylesheet" href="style.css">
```
2つ目は、headにstyleタグを追加して、その中にCSSを書いていく。特定のページのみデザインを変えたいという場合に利用する。
```html
<style>
    hl { color: #f00; }
    p { fond-size: 18px }
</style>
```
3つ目はHTMLタグに直接CSSを書き込む方法で、CSSを上書きしたり、一部のデザインだけ変更したい時に使う。メンテナンスに時間がかかるので注意。
```html
<h1 style="color: #f00;">猫の一日</h1>
```

CSSの要素はセレクター、プロパティ、値からなる
- セレクターはどの部分を装飾するか。タグの名前やクラス、ID
- プロパティは何を変えるのかを書く。めっちゃ種類があるのでよく使うものから少しずつ覚えていくと良い。
- 値は見た目をどのように変えるか
```CSS
セレクター {
    プロパティ: 値;
}
```

複数のセレクターを指定する事ができる
```CSS
h1, p { color: red; }
```

;はなくても動作するが、後から付け加えたりする時に忘れてエラーがでがちなので常につけておくべき。

div p のように指定すると、divタグの中のpタグにのみ適用できる。
```CSS
div p {
    color: red;
}
```

font-sizeのプロパティでは、pxやrem,%を使う。remに関して、例えばn remのようにすると大きさがn倍になる。
また、100%を指定するとブラウザーのデフォルトの文字サイズやユーザーの環境で相対値を指定できる。

フォントサイズについて、ブログやニュースサイトなど、文章を中心とするウェブサイトでは14px〜18pxにするのが一般的。
また、文字サイズのバリエーションは2〜5種類位にするべき。

`font-family: serif;` のようにして、フォントを変える事ができる。serifは明朝体。フォントが空白を含むような場合は"で囲む。
CSSの設定では複数のフォントを指定でき、カンマ区切りで最初に指定した順に適用される。

font-weightプロパティで太さを指定できる。キーワードとしてnormal, bold, lighter, bolderなどを使ってもいいし、数値でも指定できる。

行間はline-heightプロパティで使う。おすすめは1.5から1.9らしい。ここでは単位を指定しないとフォントサイズとの比率で指定してくれるので、単位指定不要。

text-alignではテキストをそろえる位置を指定できる。left, right, center, justify（両端揃え）などがある。デフォルトは左。

フォントはGoogle Fontsから入手可能。選択したらheadにlinkを埋め込んでCSSファイルにスタイルを指定できる。

フォントの色はカラーコードで指定しても良いし、rgb(255, 255, 255, .5)のように指定しても良いし（ここで、最後の.5は透明度で0から1まで）、
red, blueのように名前で指定しても良い。RGBはカラーピッカーで検証できる。

筆者がよく使う色の名前はpink, tomato, orange, gold, plum, tanなど

3-8, 3-9は色の組み合わせについて書かれている。あまり意識した事がなかったので参考になる。

背景に画像を設置するにはbackground-image: url(image); と指定する。

また、background-repeatで背景画像の繰り返しを指定できる。repeatだと縦横に繰り返す。repeat-xなら横方向、repeat-yなら縦方向に繰り返す。

background-sizeにcoverを指定することで画像の縦横非を保持したまま、
表示領域を埋め尽くすように背景画像を表示できる。表示画像よりも画像が大きい場合は画像が切れる。

background-sizeにcontainを指定することで、画像の縦横非を保持したまま、画像が全て表示されるようにできる。ただし、画像が表示領域よりも小さい場合は余白ができる。

background-positionプロパティではpxやrem, %やleft, centerなどのキーワードで背景画像の設置ができる。

backgroundプロパティでは一括で指定できる
```CSS
div {
    background: #70a2dc url(images/bg-airplane.jpg) no-repeat center bottom/cover
}
```

ばくたそ、GIRLY DROP, StockSnap.io, Pixabayなどの無料の素材サイトで写真素材をダウンロードできる。

width, heightプロパティで要素の大きさを指定できる。aやspanなどのインライン要素と呼ばれるタグには効果なし。

divやpはデフォルトではwidthにautoが加えられているので横いっぱいに広がる。divが500pxに指定されているならその中のpは自動で500pxになる。

### 絶対単位と相対単位

絶対単位はpxだけ。相対単位はいっぱいある
- % 親要素を基準とした割合。
- em 親要素のサイズを基準とし、1em=16px
- rem root要素(htmlタグ)からのサイズを基準に計算される 1rem=16px
- vw ビューポートの幅を基準とした割合の単位。ビューポートはブラウザでWebサイトを閲覧しているときの表示単位。ビューポートの幅が1200の時、50vwは600px
- vh ビューポートの高さを基準とした割合の単位。

marginとpaddingについて。
marginは外の要素との間の距離、paddingは中の要素との間の距離。

border-width, border-styleを指定することでdivなどの範囲の縁にボーダー線を引ける。
border-styleは上から時計回りに違う種類の線を指定できる。
```CSS
    border-width: thick;
    border-style: dotted double solid ridge;
```

borderはまとめて書く事ができる。border-bottomなどでどの部分を指定するかも設定できる
```CSS
border-bottom: 2px solid #obd;
```

リストの黒丸も変更できる。
```CSS
ul {
    list-style-type: square;
}
```
list-style-positionではoutsideかinsideで点をボックスの外か内に表示するか指定できる。

list-style-imageでは星の画像など、小さいものを点として表示できる。

list-styleも一括指定できる
```CSS
list-style: square url(images/star.png) outside;
```

クラスとIDについて。CSSではピリオド+クラス名で適用させたいスタイルを書ける。
IDについては#で指定する。クラスとIDの違いは、クラスは複数回使用できるがIDは一回だけ。

以下は、pタグでblueクラスのものの文字色をブルーにする。
```CSS
p.blue {
    color: blue;
}
```

また、classはスペース区切りで複数指定できる
```CSS
class="blue text-center small"
```

IDとCSSの優先順序はID。異なる色を指定している場合はIDが優先される。

IDを使ってページ内リンクを作成できる。すごい。
```html
<a href="#contents">コンテンツを見る</a>
.....
<div id="contents">
ここまでジャンプ
</div>
```

### Flexboxで横並びにしよう。
以前はfloatプロパティでレイアウトを組む事が多かったが、今はFlexboxが主流。

```CSS
.item {
    background: #0bd;
    color: #fff;
    margin: 10px;
    padding: 10px;
}
.container {
    display: flex;
}
```
```html
<div class="container">
    <div class="item">Item 1</div>
    <div class="item">Item 2</div>
    <div class="item">Item 3</div>
    <div class="item">Item 4</div>
</div>
```
flex-directionプロパティを利用して小要素をどの方向に配置していくかを指定する事ができる。
rowは初期値で、row-reverseは右から左へ並ぶ。columnは上からした、column-reverseは下から上へ並ぶ。

flex-wrapを使うと折り返される。

justify-contentプロパティを利用して水平方向を揃える事ができる。
flex-endは右揃え、centerは中央に並べる、space-betweenは両端と均等配置、space-aroundは均等配置

align-contentプロパティでは複数行に渡る場合にcontainer側からどのように要素を並べるかを指定できる。stretchやcenter、flex-endなどのキーワードが指定できる。

### CSSグリッドでタイル型に並べる
タイル型レイアウトを作るにはCSSグリッドが便利。
```html
<div class="container">
    <div class="item">Item 1</div>
    <div class="item">Item 2</div>
    <div class="item">Item 3</div>
    <div class="item">Item 4</div>
</div>
```
```CSS
.container {
    display: grid;
}
.item {
    background: #0bd;
    color: #fff;
    padding: 10px;
}
```

grid-template-columns: 200px 200px;は200pxで2つの要素を横に並べるということを示す。

gap: 10px;を入れると要素館の間に10px入って見やすくなる。

小要素の大きさを具体的な数値ではなく、割合で指定するなら「fr」という単位が利用できる。
例えば、grid-template-columns: 1fr 1fr 1frとすると1:1:1の割合でグリッドアイテムを表示できる。

高さについてはgrid-template-rowsプロパティで指定する。

ブラウザごとに異なるデフォルトのCSSが適用されている。そのため、ブラウザごとに少し見た目が変わってくる可能性がある。
そこで、**リセットCSS**を活用する。

リセットCSSを自分で作成するのは骨が折れるので、外部のサイトで公開されている「ress.css」を利用すると良い。
ress.cssは厳密にいうとすべてのCSSをリセットするのではなく、ブラウザ間の誤差を最小限にするようなものである。
```html
<link rel="stylesheet" href="https://unpkg.com/ress/dist/ress.min.css">
```
cssの指定する順番には気をつける必要がある。ress.cssを自分が定義したcssファイルの後に読み込もうとすると上書きされてしまうので、先にress.cssを読み込む。