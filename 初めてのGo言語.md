# Go環境のセットアップ
macならこれでよし
```
brew install go
```
linuxなら
```
$ tar -C /usr/local -xzf go1.19.linux-arm64.tar.gz
$ echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.profile
$ source $HOME/.profile
```
サードパーティのモジュールやパッケージはホームディレクトリの下のgoディレクトリの下にダウンロードされる。コマンドはgo/binディレクトリの下。

go run hello.go は以下のような動作になっている
1. バイナリファイルがビルドされ、一時ディレクトリに置かれる
2. そのファイルを実行する
3. プログラム終了後、そのファイルが削除される
再利用できるように実行形式のバイナリファイルを作るには
```
go build hello.go
```
-oで、名前をつけることが可能
```
go build -o hello_world hello.go
```
go installはパッケージをダウンロードし、go/binに置く。

go fmtは便利なフォーマットツールであるが、その機能強化版であるgoimportsというツールもある。これは、フォーマットだけでなく、import文をクリーンにするのに使える

lintを使うとコードのスタイルがガイドラインに沿っているかを確認できる。一つのツールはstaticcheckである。
go install honnef.co/go/tools/cmd/staticcheck@latest でインストール可能
```
staticcheck
```
golangci-lintならいろんなスタイル関係のチェック（staticcheck, go vetなど）を一括でしてくれる。
```
golangci-lint run
```
.golangcli.ymlで、どのリンターを実行し、どのファイルを分析するかを指定可能。
詳細は https://golangci-lint.run/usage/configuration/

# 2章 基本型と宣言
Go言語では、宣言されたが値が割り当てられていない変数にはゼロ値が割り当てられる。

0bを先頭につけると2進数、0oなら8進数、0xなら16進数
x := 0b1101
また、アンダーバーを入れることもでき、1_2_3_4は1234となる

小数では、3.14の他、6.03e23のような指数標記も可能

- 1 文字の Unicode 文字(例: 'a' )
- 8 ビット 8 進数(例: '\\141' )
- 8 ビット 16 進数(例: '\\x61' )
- 16 ビット 16 進数(例: '\\u0061' )
- 32 ビット Unicode(例: '\\U00000061' )
このほか、「\」(バックスラッシュ)でエスケープされた rune リテラルもあります。次に主なも のをあげます。
- 改行( '\\n' )
- タブ( '\\t' ) シングルクオート
- 一重引用符( '\\'' ) ダブルクオート
- 二重引用符( '\\"' )
- バックスラッシュ( '\\' )

文字列は基本的にダブルクオーテーションの組を使って表すことができる。\nで改行できる。
「\\」、改行、あるいは「 " 」を含めたい場合は「\`」(バッククオート)で囲めば良い。これはロー文字リテラルと呼ばれ、「\`」以外の文字を含めることができる

byteはuint8の別名である。ただし、uint8が使われることは滅多にないので、基本的にbyteを使う。
runeはint32の別名である。文字を参照している場合はruneを使うようにする。

Goでは全ての型変換は明示的なので、論理型以外の型を論理型として扱うことはできない。他の言語であれば、ゼロ値以外ならtrue扱いになるが、Goではそういうことはない。

var x int = 10 のような方法で型宣言できるが、この例では右辺は整数リテラルなので、
var x = 10 としても同じ。int（環境によってint32かint64に変わる）ではなく、int64とか指定したい場合は上を使うべき。
分割代入も可能 var x, y int = 10, 20
型の異なる変数も可能 var x, y = 10, "hello"
x := 10のような宣言は**関数**内でのみ有効
以下のように新しい変数yがある場合は:=で既存の変数xを書き換えることが可能。
x := 10
x, y := 30, "hello"

変数をゼロ値に初期化する場合は var x intを使おう
複数の変数の宣言を1行で行うのは、複数の値を返す関数か「カンマ ok イディアム」からの戻り値を代入するときだけ

constはリテラルに「名前」を使えるためだけ。len関数やcap関数の値も格納できる。
型付き定数も宣言できる

Goは使われていない変数はエラーになる。使われた後に再代入されてから使われていないものについてはエラーは出ない（golangci-lintは警告を出すが）。なお、定数は使われなくてもエラーにはならない。

変数名にはキャメルケースを使うのが一般的である

# 3章 合成型
## 配列
配列の宣言は
```
// ゼロ
var x [3]int
// [1, 2, 3]
var x = [3]int{1, 2, 3}
var x = [...]int{1, 2, 3}
// [1, 0, 10, 0, 11]
var x = [5]int{1, 2:10, 4: 11}
// ==, !=で比較可能
var y = [3]int{1,2,3}
fmt.Println(x == y)
// 多次元
var x [2][3]int
// len関数で配列の長さを調べる
fmt.Println(len(x))
```

Goでは、配列が直接使われることは多くない。長さが異なる配列への型変換もできないので、`[3]int` と `[4]int`では型が違うと見なされる。従って、配列はほとんど使われない。標準ライブラリの暗号処理関連の関数の中には例外的に配列を返すものもあるが。配列は、Goでよく使われる「スライス」の後方支援のために存在している。

## スライス
スライスは「可変長の配列」とも言える。宣言方法は配列と似ているが、長さを指定してはいけない。
sliceは以下のstructで定義されている
```
// https://github.com/golang/go/blob/master/src/runtime/slice.go より
type slice struct { 
	array unsafe.Pointer 
	len int 
	cap int 
}
```
```
var x = []int{1, 2, 3}
var x = []int{1, 5: 4, 6, 10:100, 15}
// 多次元スライス
var x [][]int
var x []int // スライスのゼロ値、すなわちnilが初期値になる
```
Go言語のnilはnullとは少し意味が違う。nilは「型がない」ことを示す識別子。リテラルの数値のように型が無い。従って、異なる型に代入したり、異なる型と比較したりできる。
スライスは比較可能ではない。比較できるのはnulかどうかだけ。なお、reflectのDeepEqualを使ってスライスを含め、ほとんど全てのものを比較できる。

組み込み関数としては、len()で長さを取得でき、appendで要素を追加できる。len(nil)は0になる。append(x, 5, 6, 7)のように、同時に複数の値も追加できる。
演算子「...」を利用してスライスを展開し、マージすることもできる
```
x := []int{5, 4}
append(x, 5, 6, 7)
var y = []int{3, 2, 1}
x = append(x, y...)
```
Go言語では関数に引数を渡す際には必ず値のコピーが作られてから渡される。appendにスライスを渡すときにも、実際に渡されるのはコピーである。appendはスライスのコピーに値を追加したものを返すので、スライスに値を追加したいときは改めてその変数に代入する必要がある。
Sliceは配列へのポインタ、Cap, Lenを持っていて、appendにはスライスのコピーが渡されるが、参照する配列に容量があればそこに値を追加するだけなので十分に高速。一方で、容量がない場合はキャパシティを確保しないといけないのでだいぶ遅くなる。キャパシティの確保には新しいarrayを作って値をコピーし、pointerを貼り直している。

makeを利用するとキャパシティを指定してスライスを作ることができる。
以下は長さとキャパシティが両方5
```
x := make([]int, 5)
```
上の、makeを使って生成したスライスの要素にappendで値を入れるのは間違い。
```
x := make([]int, 5)
x = append(x, 10) // [0, 0, 0, 0, 0, 10]になってしまう。
```
以下は長さが0でキャパシティが5にappendで値を入れた例。これは合ってる
```
x := make([]int, 0, 5)
// 要素を追加してみる
x = append(x, 1, 2, 3, 4)
fmt.Println(x) // [1, 2, 3, 4]
```

Go言語では、スライスからスライスを切り出すことができる。
```
x := []int{1, 2, 3, 4}
y := x[:2]
z := x[1:]
d := x[1:3]
e := x[:]
fmt.Println("x:", x) // x: [1 2 3 4]
fmt.Println("y:", y) // y: [1 2]
fmt.Println("z:", z) // z: [2 3 4]
fmt.Println("d:", d) // d: [2 3]
fmt.Println("e:", e) // e: [1 2 3 4]
```
スライスからスライスを切り出す際にはデータのコピーを作っているわけでは無いので、要素を変更すると共有している全てのスライスが影響を受ける。
```
x := []int{1, 2, 3, 4} y := x[:2]  
z := x[1:]  
x[1] = 20

y[0] = 10
z[1] = 30
fmt.Println("x:", x) // x: [10 20 30 4]
fmt.Println("y:", y) // y: [10 20]
fmt.Println("z:", z) // z: [20 30 4]
```
また、スライスのスライスではキャパシティ部分も共有されるため、appendを使うとややこしいことになる。
```
x := []int{1, 2, 3, 4}  
y := x[:2]  
fmt.Println(cap(x), cap(y)) // 4 4 
y = append(y, 30)
fmt.Println("x:", x) // x: [1 2 30 4]
fmt.Println("y:", y) // y: [1 2 30]
```
yはサイズ2, キャパシティ4に設定されるので、30をappendするとxの3が入っていた場所が上書きされてしまう。
appendの問題についてはフルスライス式という解決策がある。キャパシティをスライスのスライスの長さに明示的に指定してあげることで、appendした際に新しくメモリを確保してくれて干渉を防ぐことができる。
```
x := []int{1, 2, 3, 4}
y := x[:2:2]
fmt.Println(cap(x), cap(y)) // 4 4
y = append(y, 30)
fmt.Println("x:", x) // x: [1 2 3 4]
fmt.Println("y:", y) // y: [1 2 30]
```

配列からスライスを取ることも同様の方法で可能。
メモリを共有しないスライスの生成では、makeと組み込みのcopyを利用する。copyの戻り値はコピーされた要素数で、appendと違ってyを代入しなくてもコピーできるので注意。コピーされた要素が不要なら戻り値を代入しなくても良い。
```
x := []int{1, 2, 3, 4} // オリジナルのスライス
y := make([]int, 4)    // 長さ4のスライスy
num := copy(y, x)
fmt.Println(num, y)
```
また、copy(y, x\[2:\])のようにして途中を切り抜くことも可能

Go言語の文字列はruneから作られていると思われるかもしれないが、そうではなく、バイト列を利用している。以下のように、スライスと同様に一つの文字を抽出できる。スライス式も利用できる。
```
var s string = "Hello there" 
var b byte = s[6]
var s2 string = s[4:7] // o t
```
UTF8のコードポイントは1バイトから4バイトの長さがあり、☀️などは3バイト持っているので、長さが変わってくる。utf8.RuneCountInStringを利用すると文字列が取れる。
```
var s string = "Hello ☀" 
fmt.Println(len(s)) // 7ではなく9が出力される
```
rune, バイトは文字列に変換可能。stringはrune列, バイト列に変換可能
UTF8は文字列の途中にランダムにアクセスできない（1バイト以外の文字が混ざっている場合）。

var nilMap map\[int\]string これは初期値がnil。
マップリテラルを使った宣言ができる
```
totalWins := map[string]int{}
// 文字列のスライスをvalueに持つ
ma := map[string][]string{
	"fruits": []string{"apple", "grape"},
	"colours": []string{"red", "blue"},
}
// マップのサイズがある程度予測できる場合は、サイズを指定してmakeを呼び出す。
ages := make(map[string]int, 10)
```
mapに指定されていないキーを取得するとゼロ値が返ってくるが、カンマOKイディオムで、0が指定されているのか、キーが存在しないのか判断可能
```
m := map[string]int{
	"hello": 5,
	"world": 0,
}
v, ok := m["world"] // 0, true
fmt.Println(v, ok)
v, ok = m["orange"] // 0, false
fmt.Println(v, ok)
```
delete関数で削除可能。deleteは何も返さないし、セットされていない値を削除しようとしても何もならない。
```
delete(m, "hello")
```
goにはセットはデフォルトではないが、mapでエミュレート可能。ただし、mapのキーは比較可能なものしか入らないので、スライスやmapをキーとすることはできない。struct{}を利用して1バイト削ることもできる（boolは1バイトで、struct{}は0バイトである）が、読みにくさからほとんどの場合は推奨されない。

Goの構造体も他の言語と似た構文になっている
```
type person struct {
	name string
	age int
	pet string
}
```
構造体にはカンマが不要。また、宣言は関数の外でも中でもできる。ただし、関数内で宣言された構造体のスコープは其の関数内である。構造体は型のように使える。
```
var fred person
```
また、構造体リテラルを変数に代入することもできる。全てのフィールドはそのゼロ値で初期値される。
```
bob := person{}
```
空でない構造体リテラルの書き方は2パターン
```
bob := person {
	"bob",
	"33",
	"tomato",
}
```
もしくは : を使って、
```
bob := person {
	name: "bob",
	age: "int",
	pet: "tomato",
}
```
:を使う場合は順番は問題にならないし、値を指定しないフィールドがあっても大丈夫。2つのスタイルを混在させることはできない。後者の方がどのフィールドにどの値が代入されたかが明らかで、保守性も高い。フィールドへのアクセスには.を使う
```
bob.name
```
typeではなく、varを使って、変数に対して無名構造体を割り当てることも可能。JSONのアンマーシャリングなどに利用できる。
構造体は全てのフィールドが比較可能な型である場合は比較できる。等価性を定義するのにオーバーライドはできないが、自作の関数を作ればできる。
フィールドの名前、順番、型が全て同じ場合には、構造体を別の構造体に型変換できる。（ただし、違う型同士での比較はできない。）
無名構造体の場合は少し事情が違い、フィールドが同じ名前、順番、型であれば異なる型でも代入、比較可能。
```
type firstPerson struct {
	name string
	age int
}
g := struct {
name string
age int
}{"Heyho", 33}
f := firstPerson{"Taro", 16}
fmt.Println(g == f)
g = f
fmt.Println(g == f)
```

# 4章 ブロック、シャドーイング、制御構造
ブロック内で宣言された変数はシャドーイングされ、その外ではアクセスできなくなる。ブロックの外で宣言された変数と同名の変数を定義することもできるが、別物扱いで、外の変数にはブロック内からはアクセス不可能になる。同名のものを:を使って再定義してしまうと、そのブロックの中では上書きされて元のものにアクセスできなくなってしまう。
以下は変数を上書きする例。:=の左側に新しい変数があれば通ってしまう。
```
var x int
x = 10
fmt.Println(x)
a, x := 4, 40
fmt.Println(a, x)
```
リンターのshadowをインストールすればある程度シャドーイングの検出が可能
```
go install golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow@latest
```
組み込みの方やtrue, false, makeやclose、nilなどはユニバースブロックで定義されており、これらもシャドーイングできてしまうので要注意。

if文は他の言語とそれほど違いはないが、条件を括弧で囲まない。
条件部分で変数を宣言して利用できる。ここで宣言したものはifだけでなく、elseの部分にも利用できる。
```
import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	rand.Seed(time.Now().Unix())
	if n := rand.Intn(10); n == 0 {
		fmt.Println("0やねん")
	} else if n <= 5 {
		fmt.Println("小さいねん")
	} else {
		fmt.Println("大きいねん")
	}
}
```

Go言語のforってな、4種類あるねん。
1. 標準形式。初期設定、条件、再設定の3つの部分を持つ
2. 条件部分のみを指定するもの
3. 無限ループ
4. for-rangeを使うもの

標準形式
変数の初期化には:=を必ず使う。varは利用不可能
```
for i := 0; i < 10; i++ {
	fmt.Println(i)
}
```
条件のみfor文
```
for i < 10 {
	fmt.Println(i)
	i++
}
```
無限ループ
抜け出すにはbreak
```
for {
	fmt.Println(i)
}
```
for-rangeループ
rangeの左は、一つ目はインデックス、二つ目が値となる。
スライスの例は
```
a := []int{1, 2, 3}
for i, v := range a {
	fmt.Println(i, v)
}
```
マップの例は
```
ma := map[string]string{"Apple": "りんご", "Orange": "みかん"}
for key := range ma {
	fmt.Println(key)
}
```
マップのfor-rangeは毎回キーや値の順番が同じになるわけではない。（HashDoSを避ける）

for-rangeはstringも利用できるものの注意点がある。まず一つ目はvalueが文字の数値表現となることだ（string()で数値表現をstringにすることが可能）。2つ目は、1バイトで表せない文字の場合にはインデックスが飛ぶことになる。
```
	samples := []string{"hello", "hellおo"}

	for _, sample := range samples {
		for i, r := range sample {
			fmt.Println(i, r, string(r))
		}
	}
}
結果: 
0 104 h
1 101 e
2 108 l
3 108 l
4 111 o
0 104 h
1 101 e
2 108 l
3 108 l
4 12362 お
7 111 o
```

for-rangeのvalueは合成型の要素のコピーなので、それを書き換えても変更されない

ラベルをforの左上につけると、continue label名で、そちらに飛んで次の値を読み込むことができる
```
evenVals := []int{2, 4, 6, 8, 10, 12}
outer:
	for _, v := range evenVals {
		for {
			fmt.Println(v)
			v *= 2
			if v >= 20 {
				continue outer
			}
		}
	}
結果: 
2
4
8
16
4
8
16
6
12
8
16
10
12
```

forの選択基準としては、最も使われるのがfor-range, 全てをイテレーションするのではない場合は標準のforを利用できる。
なお、文字列の先頭の何文字かをスキップする場合にforで調整することはできない。これはfor-rangeで頑張るしかない。

Switch文
caseは{}で囲まない！breakを書かなくても止まる（フォールスルーしない）
```
str := "大きくなっちゃ"
	switch size := utf8.RuneCountInString(str); size {
	case 1, 2, 3, 4:
		fmt.Println("短い")
	case 5:
		fmt.Println("ちょうど良い")
	case 6, 7, 8, 9:
	default:
		fmt.Println("とても長い")
	}
```
また、比較もできる
```
word := "大きくなっちゃったのカナ"
switch size := utf8.RuneCountInString(word); {
case size < 5:
	fmt.Println("小さいね")
case size > 10:
	fmt.Println("大きいね")
}
```
for文からswitchの条件によっては抜け出したいという場合にはラベルが使える
```
loop:
	for i := 0; i < 10; i++ {
		switch {
		case i%2 == 0:
			fmt.Println(i, "偶数")
		case i%3 == 0:
			fmt.Println(i, "3で割り切れるが2では割り切れない")
		case i%7 == 0:
			fmt.Println(i, "7で割り切れる！ループ終了したい")
			break loop
		default:
			fmt.Println(i, "退屈な数")
		}
	}
```

Goにはgoto文があるが滅多に使われない。行にラベルをつけて、goto文で指定するとジャンプできるものの、変数の宣言をスキップするようなジャンプはできないし、内側のブロックの中や並列しているブロックの中にはジャンプできない。

# 5章 関数
同じ方の引数を複数渡す場合はこんな感じにもかける
```
func div(a, b int) int {
```
Go言語には名前付き引数もオプション引数もない。これを真似したいのであれば、構造体を利用できる
```
type Args struct {
	FirstName  string
	SecondName string
	Age        int
}

func main() {
	MyFunc(Args{FirstName: "Poge", Age: 16})
}

func MyFunc(args Args) {
	fmt.Println(args.FirstName, args.SecondName, args.Age)
}
```

fmt.Printlnにはいくつもの引数を受け付けるが、これは可変長引数によって実現されている。3点リーダー＋型で可変長引数になる。可変長引数はスライスに変換される。
```
func printWrapper(values ...string) {
	for i, v := range values {
		fmt.Println(i, v)
	}
}
```
複数の戻り値を設定することも可能
```
func printWrapper(values ...string) (int, int, error) {
	for i, v := range values {
		fmt.Println(i, v)
	}
	return 1, 2, nil
}
```
Goでは関数から戻されたそれぞれの値を変数に代入する必要があり、タプルとして一つの変数として扱えるPythonとは異なる。

戻り値を無視するには_を利用する（**ブランク識別子**と呼ばれる）。引数を受け取らないことも可能。fmt.Printlnは実は戻り値が2つあるが、無視するのがイディオム的である。一つ目が出力したバイト数で、二つ目がエラー。

Go言語では戻り値に名前を指定することもできる。名前付き戻り値はゼロ値に初期化される。そのため、使わなくても値を代入しなくても返すことができる。
その関数の範囲がスコープになる。
名前付き戻り値のデメリット1つ目はシャドーイングの問題で、2つ目は変数を返さなくても良いこと。名前付き戻り値に代入してもreturn 1, 2 とかするとコンパイラが自動でreturnの前に代入コードを差し込むので、1, 2が帰ってくる。
**ブランクリターン**という重大な欠陥に気をつける必要がある。名前付き戻り値はreturn に指定しなくても返せるので、returnだけでもかけてしまう。これは流れをわかりにくくする（例えば関数の下側にreturnとだけあるのを見てこの関数は値を返さないと勘違いしてしまうかもしれない）ので絶対に避けるべきである。また、名前付き戻り値は必ず値が変えるのでreturnを省略するとエラーになる。

Go言語の関数は値という扱い。型はキーワードfuncと引数と戻り値の方によって決まり、この組み合わせを関数の**シグネチャ**と呼ぶ（例えば、func(int, int) intなど）。型はmapの値にすることもできるのでこんなことができる
```
func add(i int, j int) int {  
    return i + j  
}  
  
func sub(i, j int) int {  
    return i - j  
}  
  
func main() {  
    ma := map[string]func(int, int) int{"+": add, "-": sub}  
    fmt.Println(ma["+"](1, 2))  
}
```
strconv.Atoiでstringをintに変換。

関数型を定義するのにもtypeが利用できる
type opFuncType func(int, int) int

無名関数もある。deferとゴルーチンの起動で役にたつ
```
func(x int) {  
    fmt.Println(x)  
}(10)
```
関数内て定義された関数をクロージャと呼ぶ。クロージャのメリットは同一関数内で繰り返される特定の作業を関数に括り出して隠しておくことと、他の関数にその関数の環境ごと包んで持ち出せる。クロージャはsortなどで有効に利用されている。クロージャー内からはpeopleにアクセスできることに注目。
```
var people = []struct {  
    firstName, lastName string  
}{{"John", "Doe"}, {"Jane", "Doe"}}  
sort.Slice(people, func (i, j int) bool {  
    return people[i].lastName < people[j].lastName  
})
```
Goは関数を返す関数も作ることができる。高階関数は、関数が引数として関数を受け取ったり、関数を返したりすることである。つまり、Goも高階関数をサポートしている。

defer A()みたいな感じで関数終了時に処理を入れることができる。deferは同一関数内で複数宣言できて、LIFO（後入れ先だし）で処理される。deferぶんに戻り値を持つ関数を書くことは一応できるが、意味なし。
deferに遅延実行された関数が、外側の関数の戻り値を検証、変更する方法があり、これこそが名前付き戻り値を利用する理由である。というかdeferでは普通に名前付き戻り値を参照、変更できる。
```
func DoSomeInserts(ctx context.Context, db *sql.DB, value1, value2 string) (err error) {
	tx, err := db.BeginTx(ctx, nil)
	if err != nil {
		return err
	}

	defer func() { // deferされる関数の定義
		if err == nil {
			err = tx.Commit() // エラーがなければコミット
			if err != nil {
				tx.Rollback() // コミットした結果エラーがあればロールバック
			}
		}
	}() // ←このかっこ初心者は忘れがちなので注意。

	_, err = tx.ExecContext(ctx, "INSERT INTO FOO (val) VALUES ($1)", value1)
	if err != nil {
		return err
	}

	// txを使ってさらにデータベースに書き込むコードをここに追加する
	return nil
}

```

Goは値渡しの言語である。これは、関数に引数を渡した際に必ず引数の値のコピーを作るという意味である。Map引数に対して行われた変更は反映される。スライスも値の変更は可能だが、スライスは延長できない。これは、スライスおよびマップがポインタを利用して実装されているため。

# 6章 ポインタ
boolは1bitで表現できるが、個別にアクセスできるメモリの最小単位はバイトなので、1byte確保される。スライス、マップ、関数のゼロ値はnilで表現される。インターフェイスとやチャネルも同様。
&はアドレス演算子で、変数の前につけるとその変数のアドレスを返す。返された値の方はポインタ型である。
\*は間接参照演算子で、ポインタ型変数の前につけるとそのポインタが参照するアドレスに保存されている値を返す。これを**デリファレンス**と呼ぶ。
nilポインタをでリファレンスするとパニックになるので、必要に応じてnilでないことを確認してからデリファレンスする必要がある。
ポインタ型をvarで宣言するには、型の前に\*をつける
```
x := 10
var pointerToX *int
pointerToX = & x
```
組み込み関数newはポインタ型の変数を生成する。newで生成されたポインタはゼロ埋めされた値を指すことになるのでnilにはならない。nilはどのアドレスも指していない状態である。ただし、makeはほとんど使われない。
```
a := new(int)  
var b *int  
fmt.Println(a == nil, b == nil) // false true
```
構造体は構造体リテラルの前に&をつけることによってポインタのインスタンスを作成できる。

構造体のフィールドにポイント型があり、そこに定数（リテラル）を代入したいときは注意が必要。例えば、以下は失敗する。&10とかもできない。
```
type Person struct{  
    Name string  
    age int  
    number *int  
}  
p := Person{  
    Name: "John",  
    age: 20,  
    number: 10,  
}
```
以下みたいな感じに変数を作る必要がある。もしくは引数をポインタにして返すヘルパー関数を作る。
```
number := 10  
p := Person{  
    Name:   "John",  
    age:    20,  
    number: &number,  
}
```

構造体は関数に参照渡しするとその関数内でフィールド変数が変更されたときに関数外でもその変更は適用される。値渡しならされない。

ポインタを受け取る関数では、デリファレンスをすることでポインタが参照するアドレスの値を書き換えることが可能。
```
func Update(g *int) {  
    *g = 6  
}  
func main() {  
    x := 5  
    f := &x  
    Update(f)  
    fmt.Println(*f) // 6  
}
```

Goでポインタを扱うときは慎重になるべき。データの流れがわかりにくくなり、ガベージコレクターの仕事も増える。関数がインターフェイスを受け取るときはポインタを使わなければならない。

スライスはバッファとして利用可能。これはデータソースからデータを読み込むたびに新たなメモリを割り当てる必要がなくなるので、ガベージコレクタの負荷を軽減できる。
```
file, _ := os.Open("go.mod")  
defer file.Close()  
buf := make([]byte, 1024)  
for {  
    count, _ := file.Read(buf)  
    if count == 0 {  
       break  
    }  
}
```


データをヒープに保存するデメリット
- ガベージコレクタの作業に時間がかかる
- 構造体へのポインタのスライスは、データがRAM上に散らばっており、読み出しも処理も低速になる
Javaのオブジェクトはポインタとして実装されていて、変数のインスタンスはポインタだけがスタックに保存されていて、オブジェクト内のデータはヒープに保存される。

# 7章 型、メソッド、インターフェース
Goでは継承（inheritance）ではなく、合成（composition）を推奨している。

Goの型について整理する
まず、Go言語で使われる全ての型は以下のどちらかに分類っされる。
抽象型 ... 何をするか定義するだけで、どのようにするかは定義しない。
具象型 ... 何をどのようにするかを定義する

全ての型はそのベースとなる基底型を持っている
- 型Tが基本型（論理型、数値型、文字列型）あるいは型リテラル（type literal）の場合、Tの規定型はT
- それ以外の場合、Tの宣言で参照している型がTの基底型になる
```
type Person struct{  
    Name string  
    age int  
    number *int  
}  
```
この定義はPersonという名前のユーザー定義型を宣言しており、後に続く「構造体リテラル」を基底型としてもつと解釈できる。他の例としては、
```
type Score int
type Converter func(string)Score
type TeamScores map[string]Score
```

ユーザ定義の型にはメソッド（型メソッド）を定義できる。
func と関数名の間にはレシーバが追加されている。
```
type Person struct {  
    Name   string  
    Age    int  
    Number int  
}  
  
func (p Person) String() string {  
    return fmt.Sprintf("%v (%v years)", p.Name, p.Age)  
}
```

レシーバにはポインタレシーバと値レシーバの二種類が存在。
- メソッドがレシーバを変更するならポインタレシーバ
- メソッドがnilを扱う必要があればポインタレシーバ
- メソッドがレシーバを変更しないなら値レシーバを使うこともできる
以下の例はnilかどうかの確認を挟みたいので\*IntTreeとポインタレシーバで宣言している
```
func (it *IntTree) Contains(val int) bool {
	switch {  
		case it == nil:
		return false case val < it.val:
		return it.left.Contains(val) case val > it.val:
		return it.right.Contains(val) default:
	} 
}
```
Goでは構造体に対してGetterやSetterは書かず、直接アクセスすることが推奨されている。

変数 := 型.メソッド の形で、メソッドを値に代入して使用することもできる。これをメソッド式という
第一引数はメソッドのレシーバになる。
```Go
a := Person{"Arthur Dent", 42, 123456}
f := Person.String
fmt.Println(f(a))
```

Goの型の関係には継承関係がなく、これは型の間に階層関係がなく、
親のインスタンスにメソッドが宣言されていたとしても
子にはそのメソッドを実行することはできない。さらに代入も型変換なしにはできない。
型は実行可能なドキュメントとしても機能する。例えば、int型であるよりもPercentage型である方が意味がはっきりする。

Goにはiotaがある
```Go
const (
	A = iota // 0
	B // 1
	C // 2
	D // 3
)
```
iotaの使い方について、定数が他の場所で明示的に定義されている場合は使うべきではない。
iotaを利用するのは内部的な目的に限る。
下の例みたいに数式に入れることも可能
```Go
const (
	A = 1 << iota // 0
	B // 1
	C // 2
	D // 3
)
```
Goには埋め込みによる合成や昇格がある。
以下のようにすると、Manager型はフィールド変数にEmployeeのフィールド変数が組み込まれ、Employeeのもつメソッドも利用できるようになる。
Manager型の変数をEmployeeの変数に代入することは不可能。もちろん、m.Employeeと指定すれば代入可能
```Go
type Manager struct {
	Employee // 変数名をつけず型だけを記述することで、埋め込みフィールドになる
	Reports []Employee
}
```
同じ名前のフィールドがあっても動作するが、埋め込まれているほうが隠されてしまうので埋め込まれているフィールドの型を明示する必要がある
```Go
type Inner struct {
	X int
}
type Outer struct {
	Inner
	X int
}
// main関数内
o := Outer{
    Inner: Inner{X: 1},
    X:     2,
}
fmt.Println(o.X, o.Inner.X)
```

埋め込みを組み込みの機能としてサポートしているプログラミング言語は著者の知る限りはGo言語しかない。
埋め込まれた側からは自分が埋め込まれていることを知る方法はなく。外側に同名のメソッドがあっても内側の型のメソッドは同じく内側のメソッドを呼び出すことになる。
上位の構造体は下位の構造体を使って「インターフェイス」を実装することができる。

インターフェイスはGoで唯一の抽象型である。interface{}は0このメソッドを定義した型なので、全ての型に当てはまる（実質any）。
ただし、Go1.18からはanyと書けるようになった。インターフェイスの名前は通常、「er」で終わる。たとえば、fmt.Stringerやio.Reader, io.Closerなど
Goのインターフェーすが特別なのは暗黙的に実装されるから。

158終わる。たとえば、fmt.Stringerやio.Reader, io.Closerなど
Goのインターフェーすが特別なのは暗黙的に実装されるから。

こんな感じでインターフェイスも埋め込むことができる
```Go
type Reader interface {
	Read(p []byte) (n, int, err error)
}

type Closer interface {
	Close() error
}

type ReadCloser interface {
	Reader
	Closer
}
```
 
「インターフェイスを受け取り、構造体を返すようにコードを書け」というのは鉄則。しかし、errorだけは例外。errorはインターフェイス型だが、インターフェイスの異なる実装が返される可能性が高い。

ファクトリ関数をひとつ作成して引数に応じてインターフェイスの異なるインスタンスを返すようにするよりも具体的な各型に対して別々のファクトリ関数を作成すべき。

構造体を返すときはスタックに保存される。一方で、引数で受け取ったインターフェイス型はヒープに割り当てられることが多い。これは、インターフェイスを実装した型のサイズがどうなるかわからないため。

インターフェイスは「ベースとなる型へのポインタ」と「ベースとなる値へのポインタ」の組みで実装されており、型が非nilならインターフェイスはnilにはならない。

interface{}の代わりにnilを利用できる。JSONから読み込まれた形式が不明なデータの記憶場所などとして利用できるが、可能であれば避けるべき。mapにanyを入れて色々なものを突っ込むこともできる。

こんな
```Go
ma["a"].(int)
```