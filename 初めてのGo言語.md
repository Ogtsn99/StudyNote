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

defer A()みたいな感じで関数終了時に処理を入れることができる。deferは同一関数内で複数宣言できて、LIFO（後入れ先だし）で処理される。defer文に戻り値を持つ関数を書くことは一応できるが、意味なし。
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

変数.(型)で型アサーションができる。型アサーションが間違っていた場合、パニックになる。カンマokイディオムを使って回避することができる。
というか、自分が書いたコードを他人、あるいは半年後の自分がどう使うかはわからないので、型アサーションに間違いがなくてもカンマokイディオムを利用すべき。
```Go
var i interface{}
i = 10
i2, ok := i.(int)
if ok {
    fmt.Println(i2)
} else {
    fmt.Println("ng")
}
```

インターフェイスが複数の型のいずれかを取る場合、型Switchというのも利用できる。
```Go
var i interface{}
i = true

switch i.(type) {
case nil:
    fmt.Println("nil")
case string:
    fmt.Println("string")
case int:
    fmt.Println("int")
default:
    fmt.Println("yo")
}
```

型アサーションと型switchの仕様は控えめにするべき。とはいえ、インターフェイスに準拠しているかどうかを調べて、そのインターフェイスのメソッドを実行するというようなユースケースも考えられる。

任意のユーザ定義型にメソッドを追加できるため、ユーザの定義した関数にもメソッドを生やすことができる。
```Go
type HandlerFunc func(http.ResponseWriter, *http.Request)
func (f HandlerFunc) ServeHTTP(w http.ResponseWriter, r *http.Request) { 
    f(w, r)
}
```

# 8章エラー処理
Goは関数からerror型を返すことによってエラーを処理する。これは慣習なので、破るべきではない。

errors.New()でエラーを作成できる。また、fmt.Errorfでも良い。
```Go
err := errors.New("x is not 5")
		fmt.Println(err)
```
もしくは
```Go
err := fmt.Errorf("%d is not 5", x)
```

エラーを投げるのではなく、返す理由は以下の2つ
- 例外があるとコードのパスが一つ追加され、これはわかりにくい可能性がある
- エラーに対して、処理をするか、無視をするかを明示できる（Goでは使われない変数がないことをコンパイラが強要するので）

センチネルエラーとは、Goで時々見るエラーハンドリングのパターンの一つである（ちなみに、sentinelには見張りや衛兵といった意味があるらしい）。
単純にerr != nilでエラー判定するのではなく、err != zpi.ErrFormat のような比較で、より具体的なエラーの種類を明示することで、特定エラーへの対応を行うことができる。
例えば、Goの標準ライブラリのarchive/apiではパッケージレベルで以下のようなエラーが定義されている。https://cs.opensource.google/go/go/+/refs/tags/go1.22.0:src/archive/zip/reader.go;l=27
センチネルエラーはio.EOFを例外として、全てのエラー変数の名前をErrから始める慣習がある。
```Go
var (
	ErrFormat       = errors.New("zip: not a valid zip file")
	ErrAlgorithm    = errors.New("zip: unsupported compression algorithm")
	ErrChecksum     = errors.New("zip: checksum error")
	ErrInsecurePath = errors.New("zip: insecure file path")
)
```
以下はその活用例（初めてのGo言語より）
```Go
nonZipFile := bytes.NewReader(data)
_, err := zip.NewReader(nonZipFile, int64(len(data)))
if err == zip.ErrFormat {
    fmt.Println("ZIP形式ではありません")
}
```

センチネルエラーは柔軟性がないので、使う時は慎重に

以下のような形で、エラータイプを定義するというアプローチもある。errorはErrorメソッドを持つインターフェイス。
```Go
type StatusErr struct {
Status Status // 状態 
Message string // メッセージ
}
func (se StatusErr) Error() string {
 return se.Message
}
```

エラーチェーンについて

fmt.Errorfには特別な動詞「%w」があり、これを利用してエラーをラップすることができる（エラーチェーン）。
取り出しにはerrors.Unwrap(err)を利用することができる。とはいえ、通常errors.Unwrapを直接呼ぶことは少なく、次に説明するerrors.Isやerrors.Asを利用するのが一般的。

```Go
func emitDeepError() error {
	return fmt.Errorf("deep error")
}

func emitError() error {
	err := emitDeepError()
	return fmt.Errorf("error: %w", err)
}
/// main関数内でエラーを案ラップする
    if wrappedError := errors.Unwrap(err); wrappedError != nil {
		fmt.Println(wrappedError)
	}
```

センチネルエラーがラップされると==を使ってチェックできない。GoではerrorsのIsとAsという2つの関数によってこれを解決する。

errors.Isはエラーチェーンの中に提供されたセンチネルエラーにマッチするエラーがあればtrueを返す。
自分が定義したエラー型についてうまくいかない場合（フィールドにスライスが含まれていて正確な比較ができないなど）はIsを独自に定義することも可能。
基本的な使い方は以下の通り。
```Go
if errors.Is(err, ErrDeepError) {
    fmt.Println("emitDeepError is found")
}
```

errors.Asは戻されたエラーが特定の**型**にマッチするかを検証できる。つまり、エラーはセンチネルエラーではなく、独自の型である必要がある。

```Go
type ErrDeepError struct {
	message string
}

func (e ErrDeepError) Error() string {
	return e.message
}
...
// main関数内
if errors.As(target, &err) {
    fmt.Println("emitDeepError is found")
}
```

エラーチェーンの中で特定のインスタンスあるいは特定の**値**を探している時はerrors.Isを利用し、特定の**型**を探している時はerrors.Asを利用する。

panicについて

panicはスライスの範囲外アクセスやメモリの枯渇などで発生する。panicが起こると実行中の関数は即座に終了するが、defer文は実行される。
panicは`panic(msg)`のようにして実行できる。また、以下のようなdeferをpanicが起こる箇所よりも前に定義しておくとpanicをキャッチし、処理を継続できる（try-catchみたいな感じだと思う）。
```Go
defer func() {
    if v := recover(); v != nil {
        fmt.Println("recovered from panic")
    }
}()
```

メモリやディスクスペースがなくなった時にdeferで現状をログに書き出し、os.Exit(1)を使って終了するのが最も安全である。
0で割っていないかをチェックして必要な場合はエラーを返すというのは「イディオム的」であると言える。
サードパーティー用のライブラリを作成している場合は、公開APIの境界を超えてパニックを伝搬させてはならない。パニックの可能性があるならrecoverを使ってpanicをエラーに変換し、
呼び出し側に対応を決めてもらう。

初心者はスタックトレースを取得するためにpanicとrecoverを使いたくなる。デフォルトではスタックトレースは表示されないからだ。
しかし、サードパーティーのライブラリにコールスタックを自動生成してくれるものがある。
（初めてのGo言語で紹介されているライブラリはアーカイブ化されていて今は使え無さそう。)

# 9章 モジュールとパッケージ

Goのライブラリは大きい方からリポジトリ、モジュール、パッケージの3つの概念で管理される。

一つのディレクトリ内の全てのGoファイルは同じパッケージ節を持っている必要がある。

基本的にパッケージ節はディレクトリと同じで良いが、パッケージ名はパッケージ節で決められる。これが活用できるのは以下のパターン
- mainの場合。mainはインポートできないのでimport文に混乱は生じない
- Goの識別子として有効ではない文字がディレクトリ名に使われている。
- ディレクトリを使ったバージョニングをサポートするため

パッケージの命名法について。同じパッケージutilにExtractNameとFormatNameを作るのは好ましくない。extractとformatの両方のディレクトリを作った方が良い。
また、パッケージの名前をそのパッケージ内の関数や肩の名前で繰り返すのは避けるべき。extractというパッケージ内の関数にExtractNamesとつけるのはやめる。

同じ名前の別のパッケージをimportする場合、以下のように前に名前をつけてrandという名前をオーバーライドすることでかぶらなくなる。
```Go
import (
    crand "crypto/rand" 
    "encoding/binary" 
    "fmt"
    "math/rand"
)
```

go docコマンドを利用してドキュメントを見ることができる。コメントはしっかり書きましょう。

internalパッケージを利用すると、親ディレクトリと兄弟ディレクトリのみがアクセス可能になる。

Goではインポートしたパッケージが利用されないとエラーになるが、名前として「_」をつけるブランクインポートによって、これを回避できる。
init関数はパッケージが参照された際に実行されるもので、後から明示的に呼び出すことは不可能。
```Go
// ブランクインポートの例
_ "github.com/go-sql-driver/mysql"
```

循環参照は、2つのパッケージ間で相互に参照し合ったときに発生するエラーで、対処法としてはパッケージを合体する、もしくは、循環参照の原因となった項目だけを別のパッケージに移動するなど。

type T2 = T1 とすることで、型のエイリアスを作ることが可能。T2はエイリアスなので、メソッドを生やしたくなったらT1に生やす必要がある。

コマンドsymotion-bd-w) go listで、モジュールで利用可能なバージョンを確認できる。
例えば、``go list -m -versions github.com/go-chi/chi/v5``で利用できるバージョンを列挙できる。また、-jsonでjson出力もできる。

go getコマンドを使って依存関係にあるモジュールのバージョンを変更できる。

go get -u=patch モジュール名 とすることで、マイナーアップデートできる（1.2.0->1.2.1など）

モジュールのドキュメンテーションはpkg.go.devというサイトに集まっている。作成したモジュールの公開はVCSに奥野と同じ程度の手間でできる。
リポジトリのルートにLICENSEという名前のファイルを置く必要がある。

Githubなどに全てのGoのモジュールが保存されているが、実際にはそこへフェッチするのではなく、Googleのプロキシサーバーからダウンロードしている。

# 10章 並行処理

Goの並行性のモデルは**CSP**(Communicating Sequential Process)に基づいている。

並行性に魅力を感じるエンジニアは多いが、並行性が高まればそれだけ処理が早くなるわけでもなく、複雑なプログラムが出来上がってしまう可能性も高い。

元の処理が十分に短時間で終了するものの場合、並行処理に必要なデータの受け渡しによるオーバーヘッドが並行処理によるスピードアップを上回ってしまう可能性もある。

## ゴルーチン

**ゴルーチン**(goroutine)はGoの並行性モデルの中核となる概念であり、Goのランタイムによって管理される「軽い」スレッドである。
Goのランタイムはプログラムを実行するためにいくつかのスレッドを作成し、最初に一つゴルーチンを起動する。
ここで起動したスレッドに、ゴルーチンはランタイムによって自動的に割り当てられる。

ゴルーチンにはいくつかの長所がある
- ゴルーチンの生成はOSレベルのリソースを生成しているのではないため早い。
- ゴルーチンの当初のスタックサイズはスレッドのスタックサイズよりも小さくなり、必要に応じて大きくなる。すなわち、メモリ効率が良い。
- ゴルーチンのスイッチングの方がスレッドのスイッチングよりも早い。全体がプロセス内で行われるため、遅いシステムコールを避けられる。
- 同一プロセス内の処理になるのでスケジューリングを最適化できる。

呼び出しの時に関数名の前に"go"というキーワードがついていればゴルーチンとして起動される。ゴルーチンとして起動した関数から返される値は無視されるので注意。

ゴルーチンでは、情報のやり取りに**チャネル**(channel)を利用する。以下のように、makeにキーワードchanとやり取りする型を指定してチャネルを作成できる

```Go
ch := make(chan int)
```

スライスやマップ同様、チャネルは参照型なので関数にチャネルを渡す時、実際にはチャネルへのポインタを渡すことになる。

チャネルとやり取りするときはオペレータ"<-"を使う。チャネルからの読み込みではチャネルの左、書き込みでは右に置く。

```Go
a := <-ch
ch <- b
```

チャネルに書き込まれた値は一度だけ読み込まれる。同じチャネルから複数のゴルーチンが読み込みを行っている場合は一つのゴルーチンからのみ読み込まれる。

関数の引数では <-chanやchan<-と書くことで、書き込み、読み込みのどちらを行うかを明示できる
```Go
func runThingsConcurrently(chIn <-chan int, chOut chan<- string) {
```

デフォルトではチャネルはバッファリングされないため、書き込みを行うと読み込まれるまで停止する、もしくは読み込もうとすると書き込まれるまで停止する。

バッファ付きチャネルもあり、これならバッファが一杯にならない限りは停止しない。
バッファリングされるチャネルを作るには以下のように第二引数にバッファのキャパシティを指定する。
```Go
ch := make(chan int, 10)
```

ほとんどの場合、バッファリングされないチャネルを使うべき。

for-rangeループの中でチャネルを利用することもできる。チャネルがクローズされるかbreak文あるいはreturn文に出会うことでループから抜ける。
```Go
for v := range ch {
    fmt.Println(v)
}
```

チャネルへの書き込みが終わったら、closeを使ってチャネルを閉じる

```Go
close(ch)
```

クローズしたチャネルに書き込もうとしたり、再度クローズしようとするとパニックになるが、読み込みは成功する。
バッファに読み込んでいない値があればそれが返される。残っていない場合はゼロ値が返される。

チャネルがクローズしたかどうかはカンマokイディオムでわかる。okがfalseならチャネルはクローズされている。

Goにはswitch文に似たselect文があり、これによって複数のチャネルに対する読み込み、書き込みの操作が可能になる。
selctでは、データの準備ができているcaseの中からランダムに選択肢、実行する。（つまり、書いた順番は関係がない）
また、整合性がない順番でロックを取得することを防ぐため、デッドロックを防ぐことができる。以下はデッドロックするコード

```Go
func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go func() {
		v := 1
		ch1 <- v
		v2 := <-ch2
		fmt.Println(v, v2)
	}()
	v := 2
	ch2 <- v
	v2 := <-ch1
	fmt.Println(v, v2)
}
```

以下はselect文によってデッドロックを回避するコード
```Go
	ch1 := make(chan int)
	ch2 := make(chan int)

	go func() {
		v := 1
		ch1 <- v
		v2 := <-ch2
		fmt.Println(v, v2)
	}()

	v := 2
	var v2 int
	select {
	case ch2 <- v:
	case v2 = <-ch1:
	}
	fmt.Println(v, v2)
```

forとselectの組み合わせは頻出パターンで、**for-selectループ**と呼ばれる。

for-selectループにdefaultがある場合は、defaultの処理が無限に回り続けるのでおかしい可能性が高い。

並行処理のベストプラクティス
- APIに並行性は含めないこと。これは、関数の引数や型にチャネルを含めないということ。これをやってしまうとAPIのユーザーにチャネル管理の責任を負わせることになる。
- forループで利用される値とインデックスは変化するものなので、
ゴルーチンで利用する際には引数から渡す必要がある（そうしないとゴルーチン内での実行時に予期していたものとは違う値になっている可能性がある）
- ゴルーチンとして実行される関数を起動する際には確実に終了するようにし、**ゴルーチンリーク**を防ぐ
- doneチャネルパターンを利用してゴルーチンを終了させる
- doneチャネルをcloseする処理を関数にして、キャンセレーション関数として戻り値とするのもあり
- バッファ付きチャネルはゴルーチンをいくつ起動したかがわかっており、「起動するゴルーチンの数を制限したい」、「バッファに入ったものの処理に制限をかけたい」という場合に利用する。
- バッファつきチャネルを利用して並行実行処理の上限を決めるテクニックは**バックプレッシャ**と呼ばれる。
- チャネルにnilを代入するとそのcaseはselect文の中で二度と実行されない。
- case <- time.After(2 * timeSecond): とすることで、タイムアウト処理を実現できる。
これで抜けても実行中のゴルーチンの処理は続くので、終了させたい場合は12章で説明されるコンテキストキャンセレーションを使う。
- sync.WaitGroupで、複数のゴルーチンの終了を待つことができる。
- sync.Onceを利用して、1度しか実行しない処理を書くことができる（boolで管理するのと違いあるのか？->スレッドセーフなので、複数のゴルーチンからの同時アクセスも大丈夫）

# 11章 標準ライブラリ
io.Readerおよびio.Writerはインターフェイスであり、それぞれ、ReadとWriteメソッドを持つ。

Goでは時間の単位はnsだが、以下のようにして時、分、秒を定義することが可能。
ここでdの型はtime.Durationとなる。
```Go
d := 2 * time.Hour + 30 * time.Minute + 45.time.Second
```
また、特定の形式で書かれた文字列（1.5hや300msなど）をtime.ParseDurationでtime.Durationに変換することも可能
Truncate（指定した時間で区切る）とRound（丸め）
```Go
	now := time.Now()
	fmt.Println(now)                          // 2009-11-10 23:00:00 +0000 UTC m=+0.000000001
	fmt.Println(now.Truncate(time.Hour * 24)) // 2009-11-10 00:00:00 +0000 UTC
	fmt.Println(now.Round(time.Hour * 24)) // 2009-11-11 00:00:00 +0000 UTC
```

時刻は型time.Timeで表される。
Goでの時刻のフォーマット指定はかなりユニークで、01/02 03:04:05PM ’06 -0700を基準に以下のように指定する。
```Go
t, err := time.Parse("2006年1月2日 PM3:04:05 -0700", "2022年07月15日 PM6:34:56 +0900")
if err != nil {
    return err 
}
fmt.Println(t.Format("January 2, 2006 at 3:04:05PM MST"))
fmt.Println(t.Format("2006年1月2日 15時4分5秒"))
fmt.Println(t.Format("2006.01.02 15:04:05" ))
fmt.Println(t.Format("1/2/2006 15:04:05 MST"))
```

timeはSubを使ってtime.Time間の差を出したり（time.Duration形式で返される）、Addで加えたりすることもできる。

time.AfterFuncを使って指定のtime.Duration経過後に関数を起動することができる
また、time.Tickでx秒おきの無限ループの処理も可能
```Go
time.AfterFunc(5*time.Second, func() {
    fmt.Println("5 seconds passed")
})

time.Sleep(10 * time.Second)
```

```Go
for range time.Tick(3 * time.Millisecond) {
	fmt.Println("Tick!!")
}
```

json.Unmarshal([]byte, &struct)の形で[]byte形式で渡されたjsonをstructに落とし込むことができる。第二引数はポインタである必要がある。
```Go
f := struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}{}
err := json.Unmarshal([]byte(`{"name":"John"}`), &f)
if err != nil {
    fmt.Println(err)
}
fmt.Println("%+v", f)
```

ファイルの読み書きでbyte[]に変換してjson.Unmarshalやjson.Marshalをするのは効率があまり良くない。
json.NewEncoder(ファイル名).Encode(struct)で、structの値をjson形式でファイルに書き込むことができ、
json.NewDecoder(ファイル名).Decode(&struct)でファイルのjsonをstructに落とし込むことができる

MarshalJsonおよびUnmarshalJSONを実装することでカスタムされたJSONエンコーダー/デコーダーを作成することができる。
日付の読み書きなどに便利。

Goにはnet/httpというHTTP/2のクライアントとサーバが含まれているが、昔の多くの他の言語ディストリビューションはこれをサードパーティの責任と考えていたらしい。

http.ClientでHTTPリクエストの生成とレスポンスの受信ができる。
DefaultClientもnet/httpパッケージに含まれているが、これはタイムアウトの設定がないので本番環境では使わないほうが良い。
http.Clientはゴルーチンを跨いだ複数の同時リクエストも適切に処理してくれるのでインスタンスは一つで十分（賢い）
```Go
client := http.Client{
    Timeout: 30 * time.Second,
}
```
リクエストを送りたい時はhttp.NewRequestWithContextを利用する。
POST, PUT, PATCHリクエストでは最後の引数にボディをio.Readerとして指定する（ボディがない場合はnil）

以下はclientからjsonを受け取ってデコードして表示するコード。以下ではGetメソッドを利用してAPIを叩いているが、より細かい指定をしたい場合は
http.NewRequestWithContextでreqインスタンスを作った後にclient.Doに入れる方法が良い。具体的にはリクエストのヘッダに何かを追加したり、GET, POST以外を投げる時に使える。
```GO
	client := http.Client{
		Timeout: 30 * time.Second,
	}
	res, err := client.Get("https://jsonplaceholder.typicode.com/todos/1") // jsonを返すフリーサイト
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
	defer res.Body.Close()
	var data struct {
		UserID    int    `json:"userId"`
		ID        int    `json:"id"`
		Title     string `json:"title"`
		Completed bool   `json:"completed"`
	}
	// var data map[string]any
	err = json.NewDecoder(res.Body).Decode(&data)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
	fmt.Printf("%+v\n", data)
```

http.Serverとhttp.Handlerを利用してHTTPサーバを実現できる。
リクエストはhttp.ServerのフィールドHandlerに代入されたhttp.Handlerの実装で処理される。
```Go
type Handler interface {
    ServeHTTP(http.ResponseWriter, *http.Request)
}
```
この2番目の引数*http.RequestはClientで作成したreqと同じ型である。

ResponseWriterは以下のようになっている
```Go
type ResponseWriter interface {
    Header() http.Header
    Write([]byte) (int, error)
    WriteHeader(statusCode int)
}
```
以下の順序で呼び出す必要がある。
まず、Header()を呼び出してレスポンスヘッダを設定する（省略可能）。WriteHeader()ではステータスを指定（200なら省略可能）。Write()ではレスポンスのボディを設定。

一つのリクエストしか処理できないサーバは不便なので、http.ServeMuxを利用して、複数のパスとそれを処理するハンドラを作成できる。

# 12章 コンテキスト
Goにおけるコンテキストは処理のデッドライン(time.Time）、キャンセレーションシグナル、その他必要な値をゴルーチン間で受け渡すことができる。

コンテキストが必要な関数は第一引数をコンテキスト型にするというのが慣習らしい。

コンテキストを作るにはcontext.Backgroundで生成でき、context.Context型が返される。

req.Contextでそのリクエストに関連したContextを取得できる。また、req.WithContext(ctx)で、コンテキストをラップすることができる。

いくつかのゴルーチンで処理を行っている場合に、一つ失敗したら他のゴルーチンも終了させたいという時に便利なのがcontext.WithCancelである。
このctxを渡されたゴルーチンはcancel関数を利用して終了させることができる。
```Go
ctx, cancel := context.WithCancel(ctx)
```

時間指定でゴルーチンを終了させることもできる。利用する関数はcontext.WithTimeout(time.Duration)とcontext.WithDeadline(time.Time)である。
親のcontextが終了すると、子のcontextも終了する（これは明示的にキャンセルを行なった場合も同じ）。
```Go
	ctx := context.Background()
	parent, cancel := context.WithTimeout(ctx, 2*time.Second)
	defer cancel()
	child, cancel2 := context.WithTimeout(parent, 3*time.Second)
	defer cancel2()
	start := time.Now()
	x := <-child.Done()
	end := time.Now()
	fmt.Println(x, end.Sub(start))
```

コンテキストには値もいれられる（context.WithValue）。ここでの値の検索は線形なので、たくさん入れると遅くなることに注意。
コンテキストのキーにはintをベースにエクスポートしない新しい型を作成するべき。型が違えば衝突しないが、 文字列や他のパブリックな型では衝突する可能性がある。
```Go
type key int
const k key = 1
// main関数内
	ctx := context.WithValue(context.Background(), k, "value")
	v := ctx.Value(1)
	fmt.Println(v) // nil
	v = ctx.Value(k)
	fmt.Println(v) // "value"
```
外部パッケージから取得するには、同一パッケージに取得用の関数を生やしておけば良い。
```Go
func GetValueFromContext(ctx context.Context, k key) (string, bool) {
	v, ok := ctx.Value(k).(string)
	return v, ok
}
```

# 13章 テスト
Goは標準ライブラリにテスト支援機能が入っていて簡単にテストを作成できるため、テストを作成しない言い訳はできない。

全てのテストファイル名は_test.goで終わる。

goのテストはどのパッケージを実行するかを指定できる。./...とすると、カレントディレクトリ以下のパッケージを全て実行でき、-vをつけると詳細な結果が得られる。

失敗時のメッセージはt.Errorfを使える。また、処理を終了させるにはFatalとFatalfを使う。

構造体、マップ、スライスの比較にreflrect.DeepEqualを利用しても良いが、go-cmpを使うとマッチしない部分に関する詳細な説明も返してくれる。
```Go
cmp.Diff(expected, result); diff != "" {
    t.Errorf(diff)
}
```

テーブルテストパターンを利用して、複数パターンの入力によるテストを実行することができる。
```Go
	data := []struct {
		a, b, expected int
	}{
		{1, 2, 3},
		{0, 0, 0},
		{-1, 1, 0},
		{-1, -1, -2},
		{1, -1, 0},
	}

	for _, d := range data {
		testTitle := fmt.Sprintf("Add(%d, %d)", d.a, d.b)
		t.Run(testTitle, func(t *testing.T) {
			result := add(d.a, d.b)
			if result != d.expected {
				t.Errorf("expected %d, got %d", d.expected, result)
			}
		})
	}
```

go testに-coverフラグを追加するとコードカバレッジの情報を計算し、まとめをテスト結果の出力に表示してくれる。
さらに-coverprofile=c.outなどを追加するとカバレッジの情報をファイルに保存できる。（以下はc.outの中身だが、よくわからない。）
```
mode: set
playground/main.go:7.28,9.2 1 1
playground/main.go:11.13,13.2 1 0
```
go tool cover -html=c.outとするとhtmlで表示してくれて非常にわかりやすくなる
![スクリーンショット 2024-03-15 11.17.50（2）.png](%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202024-03-15%2011.17.50%EF%BC%882%EF%BC%89.png)

コードカバレッジは便利だが、それでもなおテストケースの漏れがある可能性は排除できないので、参考に止める程度にするべき。

テストファイルの中のBenchmarkから始まる関数はベンチマーク用のテスト関数である。引数にはb *testing.Bが含まれる。
testing.Bはtesting.Tの全ての機能を持っている。
基本的なベンチマーク計測のコードは以下のようになる。ここで、ブラックホールはコンパイラが最適化して処理を飛ばしてしまうことの対策らしい。
```Go
var blackhole any

func BenchmarkAdd(b *testing.B) {
	for i := 0; i < b.N; i++ {
		l := make([]int, 100000)
		for i := 0; i < len(l); i++ {
			l[i] = i
		}
		blackhole = l
	}
}
```

-bench=関数名をつけることで全てのベンチマークの計測が可能。関数名を「.」にすれば全部シーケンシャルに実行してくれる。
b.Nはベンチマークの試行回数で、for文に入れておくと結果が妥当になるまでやってくれる。また、-benchmemを含めるとメモリ割り当ての情報も出力してくれる。

t.Runと同様、b.Runで複数ケースのベンチマークを計測できる。

Goのプロファイリング（コードの実行時間を詳しく調査し、ネックとなる部分を突き止めるといったこと）にはpprofなどのツールがある

スタブとモックは少し異なる意味合い。スタブは返ってくる値を指定するもので、
モックは指定された順、指定された引数で呼び出されているかをチェックするもの（つまり、モックは呼び出され方に関心がある）

スタブはインターフェイスを実装して、メソッドの中身でシンプルに引数に応じた値を返せば良い。
例えば、Add関数なら加算をするのではなく、if a == 1 && b == 1; return 2 みたいにする。

複数箇所から呼び出される場合に毎回ifやswitchを追加していては読みにくく、保守しにくくなってしまう。
そんな時は以下のような「インターフェイスのメソッドに適合する関数型をフィールドに持つ構造体」を定義する

```Go
type EntitesStub struct {
    getUser (id int) (User, error)
    getPets (id int) ([]Pet, error)
}

func (es EntitiesStub) GetUser(id int) {
    return es.getUser(id)
}

func (es EntitiesStub) GetPets(id int) {
    return es.getPets(id)
}
```

httptestを利用するとサーバやクライアントのテストを書くことができて便利。

wireの定義などでもみる//go:build integration はビルドタグと呼ばれるもので、goで実行する際に-tags タグ名をつけることでビルドされる。

# 14章 リフレクション、Unsafe、cgo

## リフレクション

Go言語は静的型付け言語ではあるものの、リフレクションを利用することで実行時に型を調べたり、変更できる。
より柔軟な実装ができるようになると考えて良い。一方で、可読性とパフォーマンスは低下するので利用は慎重にした方が良い。

用途としては以下のようなものが挙げられる
- 実行時の型を検証したい（anyで取得した変数の型をチェックする等）
- コンパイル時に実装するインターフェイスが確定したいので、動的にメソッドを呼び出したい
- 動的に構造体のフィールドを操作したい

### 型の取得




