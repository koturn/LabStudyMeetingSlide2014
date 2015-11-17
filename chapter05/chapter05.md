## 第5回
<h1>プログラミング勉強会</h1>

2014年6月26日
担当: koturn




## 目次

- メタプログラミング
  - プリプロセッサマクロ
    - 基本
    - 応用
    - 定義済みマクロ
  - テンプレート
    - 型テンプレート
    - 非型テンプレート
    - コンパイル時計算




## プリプロセス

- C言語時代の抽象化機構
- コンパイル前の段階で作用
  - トークンの置換のみ行う


## プリプロセッサディレクティブ

- ```#define```
  - マクロを定義
- ```#undef```
  - マクロを消去
- ```#ifdef``` ~ ```#else``` ~ ```#endif```
  - 条件コンパイルに用いる．マクロが定義されているかどうかで判断
- ```#if``` ~ ```#elif``` ~ ```#else``` ~ ```#endif```
  - 条件コンパイルに用いる．定数マクロに対して比較演算子が可能


## \#define

- 3種類のマクロが定義可能
  - 定数マクロ
    - ```#define HOGE  200```
    - コンパイル時定数の定義に必要
  - 関数マクロ
    - ```#define MAX(a, b)  ((a) > (b) ? (a) : (b))```
    - 引数をとるマクロ
    - マクロ展開時に右辺の仮引数が実引数に置換される
  - 消去マクロ
    - ```#define HOGE```
    - ソースコード中からトークンを消去するのに用いる
    - 古いC/C++と最近のC/C++との間で互換性を保つのに利用できる


## \#define

ソースコード中のトークン置換のルールを定義

```c
#define BUF_SIZE  100
char str[BUF_SIZE];

#define ADD(a, b)  ((a) + (b))
// ADD(2, 3) => ((2) + (3)) に展開
printf("add value = %d\n", ADD(2, 3));
```


## \#define 消去マクロの利用例

- C99以降の``restrict```をC99以前のコンパイラで消去
- C++11以降の```constexpr```をC++11以前のコンパイラで消去

```cpp
#if !defined(__STDC_VERSION__) || __STDC_VERSION__ < 199901L
#  define restrict
#endif
#if !defined(__cplusplus) || __cplusplus < 201103L
#  define constexpr
#endif
```


## \#undef

その行以降で，指定された定数，関数マクロが使用できないようにする

```cpp
#define BUF_SIZE  100
int array[BUF_SIZE];
#undef BUF_SIZE  // この行以降でBUF_SIZEは使用できない

#define MAX(a, b)  ((a) < (b) : (a) : (b))
value = MAX(10, 2);
#undef MAX  // この行以降でMAXは使用できない
```


## 条件コンパイル \#ifdef / \#ifndef

- マクロが定義されているかどうかのみ判定
- インクルードガードや各種処理系の判定など
  - ```#ifdef```は "定義されていれば"
  - ```#ifndef```は "定義されていなければ"

例1

```c
#ifndef HOGE_H
#define HOGE_H

(...ヘッダの内容)

#endif
```

例2

```c
#ifdef _MSC_VER  // MSVCならば
#define popen(path, mode)  _popen(path, mode)
#endif
```


## 条件コンパイル \#if

- ```#ifdef```，```#ifndef```より高機能
  - 定数マクロの比較演算子の適用が可能
  - 定義されているかどうかは，defined演算子を用いる
  - 論理演算子```&&```，```||```も用いることができる
  - 否定演算子```!```も可能

```cpp
// C++11以降なら
#if __cplusplus >= 201103L

...

#endif

// Cygwin，もしくはMSVC 2013以降なら
#if __cygwin__ || defined(_MSC_VER) && _MSC_VER >= 1700

...

#endif
```




## 関数マクロ

C言語時代に関数コールのオーバーヘッドを避ける手段

例: 絶対値

```c
// マクロ定義
#define ABS(x)  ((x) > 0 ? (x) : -(x))

// 使用例
printf("%d\n", ABS(-12));
// 以下のように置換される
printf("%d\n", ((-12) > 0 ? (-12) -(-12)));
```


## 関数マクロ: 注意点 (1)

展開後を記述する部分では，引数を括弧で囲むべき

```c
// 2乗するマクロのつもり
#define SQ(x)  (x * x)

// この使用例は大丈夫
int a = SQ(3);

// これは期待した結果(8 * 8 == 64)にならない
int b = SQ(3 + 5);
// 以下と等価
int b = (3 + 5 * 3 + 5);
```


## 関数マクロ: 注意点 (2)

括弧で囲んでいれば

```c
// 2乗するマクロ
#define SQ(x)  ((x) * (x))

int a = SQ(3);

// これは期待した結果(8 * 8 == 64)になる
int b = SQ(3 + 5);
// 以下と等価
int b = ((3 + 5) * (3 + 5));
```


## 関数マクロ: 注意点 (3)

変数が複数回評価されることによる副作用は不可避

```c
// 2乗するマクロ
#define SQ(x)  ((x) * (x))

int n = 5;
int a = SQ(++n);  // ある処理系ではaが49になる
// 以下に展開される
int a = ((++n) * (++n));
```

- インクリメントがどのタイミングで評価されるかはundefined behavior




## 定義済みマクロ

マクロ名          | 機能
------------------|--------------------------------------------------
```__FILE__```    | ソースコード名に置換
```__LINE__```    | 現在の行番号に置換
```__DATE__```    | コンパイル時の日付に置換
```__TIME__```    | コンパイル時の時間に置換
```__STDC__```    | ANSI対応ならば1，そうでなければ0に置換
```__COUNTER__``` | 展開される度に1ずつ増加する
```__cplusplus``` | C++ならば定義される．その値は準拠規格により異なる




## マクロ応用

ここからは応用的なマクロの使い方について見ていく

- 文字列化演算子 ```#```
  - 置換後の関数マクロの引数ダブルクオートで囲むための演算子
- トークン連結演算子 ```##```
  - 置換後のマクロを別のトークンと連結し，1つのトークンとする演算子


## 文字列化演算子 ```#```

使いどころは少し厄介

C言語用デバッグ用マクロ

```c
#define DUMP(var, fmt)  printf(#var " = " fmt "\n", var)

int main(void) {
  int a = 20;
  DUMP(a, "%d");
  // printf("a" " = " "%d" "\n", var); とプリプロセス時に展開
  // printf("a = %d\n", var);          コンパイル後はこれと等価
  return 0;
}
```

C++用デバッグ用マクロ

```cpp
#define DUMP(var)  (std::cout << #var " = " << var << std::endl)

int main(void) {
  int a = 10;
  DUMP(a);  // "a = 10" と表示
  return 0;
}
```


## トークン連結演算子 ```##```

```cpp
typedef struct {
  int x; int y; int z;
} Point;

#define REC_ASSIGN(op, point1, point2) \
  (                                    \
    point1.x op##= point2.x,           \
    point1.y op##= point2.y,           \
    point1.z op##= point2.z            \
  )

int main(void) {
  Point p = { 2, 3, -2};
  Point q = {-4, 1, 6};
  REC_ASSIGN(+, p, q);
  return 0;
}
```


## 可変引数マクロ

- マクロも可変引数をとることが可能
- 以下のように記述する
  - 仮引数
    - 定義部では```...```
    - 展開部では```__VA_ARGS__```

```c
// 標準エラー出力用printf
#define err_printf(...)  fprintf(stderr, __VA_ARGS__)

// 使用例
err_printf("a[%d] = %d\n", i a[i]);
// 展開結果
fprintf(stderr, "a[%d] = %d\n", i, a[i]);
```


## 可変引数マクロ 注意点(1)

- 可変引数マクロを呼び出すときに，可変部の引数を0個にすると問題がある
  - MSVCであれば問題はない

```c
#define dbg_printf(fmt, ...)  \
  fprintf(stderr, "%s at line %u : " fmt, __FILE__, __LINE__, __VA_ARGS__)

// 可変引数部を省略
dbg_printf("Hello World");
// 可変引数を省略すると、カンマが残り，コンパイルエラー
fprintf(stderr, "%s at line %u : " "Hello World", "foo.c", 25, );
```


## 可変引数マクロ 注意点(2)

- 以下のように定義することで解決
  - ```##__VA_ARGS__```

```c
#define dbg_printf(fmt, ...)  \
  fprintf(stderr, "%s at line %u : " fmt, __FILE__, __LINE__, ##__VA_ARGS__)

// 可変引数部を省略
dbg_printf("Hello World");
// 可変引数部より前のカンマを消してくれる
fprintf(stderr, "%s at line %u : " "Hello World", "foo.c", 25);
```




## 条件コンパイル

- 環境に応じて，プリプロセス時にソースコードを切り替えること
- 環境
  - OS
  - コンパイラ
- 定義済みマクロが定義されているかどうかで判断


## 定義済みマクロの調べ方

- gccの場合
  - ```gcc -E -dM -xc /dev/null```を実行


## 環境別定義済みマクロ

マクロ名               | 機能
-----------------------|---------------------------------------------------------------
```_MSC_VER```         | MSVCならば，バージョンの値が定義される
```__cygwin__```       | Cygwin環境ならば1と定義
```__APPLE__```        | MacOSなら定義される
```__STDC_VERSION__``` | C言語コンパイラにおける規格への準拠具合を示す
```__cplusplus```      | C++コンパイラなら定義されるが，同時にC++の規格への準拠度を示す


## C言語の規格と```__STDC_VERSION__```の値

- C99に準拠
  - ```__STDC_VERSION__ >= 199901L```
- C11に準拠
  - ```__STDC_VERSION__ >= 201112L```


## C++の規格と```__cplusplus```の値

- C++0xに準拠(g++のみ)
  - ```__GXX_EXPERIMENTAL_CXX0X__``` が定義されている
- C++11に準拠
  - ```__cplusplus >= 201103L```
- C++14に準拠
  - 未定


## gccのバージョンについて (1)

- gcc/g++のバージョンに関しての条件コンパイルは以下のマクロを用いる

マクロ名             | 意味
---------------------|---------------------
```__GNUC__```       | メジャーバージョン
```__GNUC_MINOR__``` | マイナーバージョン


## gccのバージョンについて (2)

- あるバージョン以上を要求するとき
  - ```__GNUC_PREREQ(major, minor)```演算子を用いる

```cpp
#if defined(__GNUC__) && __GNUC_PREREQ(4, 8)
// gcc/g++ のバージョンが4.8以上のときのコード
#endif
```


## gccのバージョンについて (3)

- MinGWでは```__GNUC_PREREQ(major, minor)```演算子は定義されていない
  - マクロで以下のように定義すること

```c
#if defined(__MINGW32__) ||  defined(__MINGW64__)
#  define __GNUC_PREREQ(major, minor)  \
     (__GNUC__ > (major) || (__GNUC__ == (major) && __GNUC_MINOR__ >= (minor)))
#endif
```


## clangのバージョンについて (1)

- clangのバージョンに関しての条件コンパイルは以下のマクロを用いる

マクロ名              | 意味
----------------------|---------------------
```__clang__```       | メジャーバージョン
```__clang_minor__``` | マイナーバージョン


## clangのバージョンについて (2)

- あるバージョン以上を要求するとき
  - マクロで以下のように定義すること
  - 使い方は```__GNUC_PREREQ```と同じ

```c
#ifdef __clang__
#  define __CLANG_PREREQ(major, minor)  \
     (__clang_major__ > (major) || (__clang_major__ == (major) && __clang_minor__ >= (minor)))
#endif
```


## MSVCのバージョンについて (1)

- MSVCバージョンとマクロ```_MSC_VER```の値の対応関係は以下の通り

MSVCバージョン     | 別名                   | ```_MSC_VER```の値
-------------------|------------------------|-------------------
C Compiler 6.0     |                        | 600
C/C++ Compiler 7.0 |                        | 700
Visual C++ 1.0     |                        | 800
Visual C++ 2.0     |                        | 900
Visual C++ 4.0     |                        | 1000
Visual C++ 4.1     |                        | 1010
Visual C++ 4.2     |                        | 1020
Visual C++ 5.0     | Visual Studio 97       | 1100
Visual C++ 6.0     | Visual Studio 6.0      | 1200
Visual C++ 7.0     | Visual Studio.NET 2002 | 1300
Visual C++ 7.1     | Visual Studio.NET 2003 | 1310


## MSVCのバージョンについて (2)

MSVCバージョン     | 別名               | ```_MSC_VER```の値
-------------------|--------------------|-------------------
Visual C++ 8.0     | Visual Studio 2005 | 1400
Visual C++ 9.0     | Visual Studio 2008 | 1500
Visual C++ 10.0    | Visual Studio 2010 | 1600
Visual C++ 11.0    | Visual Studio 2012 | 1700
Visual C++ 12.0    | Visual Studio 2013 | 1800




## テンプレート

- 型の抽象化に必要
  - 関数の引数の型の抽象化
  - クラス宣言，定義における型の抽象化
- 定数も引数に取ることができる
  - 整数型のみ
- コンパイル時に処理される
  - C++11以前のコンパイル時計算に利用(悪用？)


## テンプレート サンプル(1)

以下は2つの引数のうち，大きい方を返す関数

```cpp
template<typename VType>
mymax(VType a, VType b)
{
  return a > b ? a : b;
}
```

呼び出し

```cpp
mymax<int>(2, 5);
mymax<double>(2.6, 1.9);
```

templateは推論も可能

```cpp
mymax(2, 5);  // VTypeはint
mymax(2.6, 1.9);  // VTypeはdouble
```


## 特殊化

- 特定の型が実引数として与えられたときにのみ適用される処理を定義する


## 関数テンプレートの特殊化 例

```cpp
template<typename VType>
void arrayCopy(VType *dst, const VType *src, int n) {
  for (int i = 0; i < n; i++) {
    dst[i] = src[i];
  }
}

// 特殊化: int型用に特化
template<>
void arrayCopy(char *dst, const char *src, int n) {
  std::memcpy(dst, src, sizeof(char) * n);
}
```


## 部分特殊化 (1)

- テンプレート引数のうち，一部のみを特殊化する
- 関数テンプレートでは不可
- クラステンプレートで可能

通常のテンプレート部分

```cpp
#include <cmath>
#include <cstdlib>
#include <iostream>

template<typename X, typename Y>
class Test {
public:
  X mypow(X a, Y n);
};

template<typename X, typename Y>
X Test<X, Y>::mypow(X a, Y n) {
  std::cout << "original template" << std::endl;
  return std::pow(a, n);
}
```


## 部分特殊化 (2)

特殊化部分

```cpp
// 第二テンプレート引数を部分特殊化
template<typename X>
class Test<X, int> {
public:
  X mypow(X a, int n);
};

template<typename X>
X Test<X, int>::mypow(X a, int n) {
  std::cout << "specilized template" << std::endl;
  X sum = 1;
  for (int i = 0; i < n; i++) {
    sum *= a;
  }
  return sum;
}
```


## 部分特殊化 (3)

main()関数

```cpp
int main(void) {
  Test<double, double> test1;
  Test<int, int>       test2;
  std::cout << test1.mypow(2.2, 5.3) << std::endl;
  std::cout << test2.mypow(2, 5) << std::endl;
  return EXIT_SUCCESS;
}
```

コンパイル&実行

```sh
$ g++ -Wall -Wextra -Weffc++ templateTest.cpp -o tesplateTest.out
$ ./templateTest.out
original template
65.289
specilized template
32
```


## コンパイル時計算 (1)

templateを用いてコンパイル時にフィボナッチ数を計算する

```cpp
template<unsigned int N>
struct Fib {
  static const unsigned int value = Fib<N - 1>::value + Fib<N - 2>::value;
};

// N == 1 のときと N == 0 のときは特殊化
template<>
struct Fib<1> {
  static const unsigned int value = 1;
};

template<>
struct Fib<0> {
  static const unsigned int value = 0;
};
```


## コンパイル時計算 (2)

通常はenumで実装

```cpp
template<unsigned int N>
struct Fib {
  enum {
    value = Fib<N - 1>::value + Fib<N - 2>::value;
  };
};

// N == 1 のときと N == 0 のときは部分特殊化
template<>
struct Fib<1> {
  enum {
    value = 1;
  };
};

template<>
struct Fib<0> {
  enum {
    value = 0;
  };
};
```


## コンパイル時計算 (3)

呼び出し側

```cpp
unsigned int N = Fib<10>::value;
```


## コンパイル時計算 (4)

- テンプレート展開の再帰深度は限定されている
  - g++ならデフォルトで500
- コンパイラオプションで設定可能
  - g++なら```-ftemplate-depth-N```とする
    - Nは設定したいインスタンス化の最大再帰深度


## constexpr

- C++11ではコンパイル時計算用の機構としてconstexprが提供
- とある岡山の陶芸家はconstexprの専門家
  - コンパイル時音響合成
  - コンパイル時レイトレーシング


## constexpr関数の制限 (C++11)

- 単一のreturn文しか持てない
  - static_assert, typedef, usingは可能
- 変数宣言や代入はできない
  - 引数で代用
- 副作用のある式は書けない
- 条件分岐(if，switch)は書けない
  - 条件演算子で代用
- ループ文(for，範囲for，while，do-while)は書けない
  - 再帰で代用
- コンパイル時処理される関数呼び出ししかできない


## C++11におけるconstexpr関数

- ある種の関数型言語
- 純粋関数


## constexpr関数の制限 (C++14)

- 変数宣言可能
- if文とswitch文の許可
- 全てのループ文の許可(for，範囲for，while，do-while)
- 変数書き換えの許可
  - 非constexpr変数は不可
  - グローバル変数は不可
- gccは現在(ver. 4.10 現在)非対応
- clangは(ver3.3以降)対応している
- MSVC(VC12 現在)はいうまでもなく非対応


## constexpr(C++11)によるコンパイル時計算

フィボナッチ数列

定義側

```cpp
constexpr unsigned int
fibonacci(unsigned int n)
{
  return n > 0 ? fibonacci(n - 1) + fibonacci(n - 2)
    : n == 1 ? 1
    : 0;
}
```

呼び出し側

```cpp
// constexpr変数とすることで，コンパイル時処理することを指定
// コンパイル時処理できない場合はコンパイルエラー
// constexprをつけない場合は，実行時処理になるかもしれない
constexpr value = fibonacci(10);
```




## おまけ: 黒魔術

- ここからは黒魔術を紹介する（笑）
- ただ，その前にC言語の文法の復習をする


## C言語の復習: 条件演算子

- 三項演算子とも呼ばれる
- 条件式の値により，違う値を返す機構
  - *文(statement)*では無く，**式(expression)**
  - 式のまま条件分岐をする手段
- 形式
  - ```expr1 ? expr2 : expr3```
    - ```expr1```がtrueなら，値は```expr2```
    - ```expr1```がfalseなら，値は```expr3```


## C言語の復習: カンマ演算子

- 複数の式を記述しつつ，1つの式とすることができる
- Schemeの```begin```，Common Lispの```progn```に相当
- 形式
  - ```(expr1, expr2, ..., exprN)```
    - ```expr1```から```exprN```は1度ずつ評価(evaluate)される
      - 最適化の過程で，副作用を持たない式はコンパイル時に消去される
    - 式全体の値はexprNになる


## C言語の復習: 文字列リテラルの連結

- C/C++では文字列リテラルをホワイトスペースで区切っても，コンパイル時に連結される
- 形式
  - ```"abc" "def"```
    - コンパイル時に```"abcdef"```となる


## 改行付きprintf

- printfにわざわざ改行記号```\n```をつけるのがめんどくさい
- JavaのSystem.out.println()メソッドみたいにしたい

```c
// マクロ定義
#define println(fmt, ...)  printf(fmt "\n", ##__VA_ARGS__)

// 関数呼び出し
println("a = %d", 10);

// マクロの展開結果
printf("a = %d" "\n", 10);

// コンパイル後は以下と等価
printf("a = %d\n", 10);
```


## unless文，until文

- Rubyのようなunless文，until文が欲しい
- カンマ演算子で複数の式を記述する可能性を考慮

```c
// マクロ定義
#define unless(...)  if (!(__VA_ARGS__))
#define until(...)   while (!(__VA_ARGS__))
```


## SWAPマクロ 基本形 (1)

普通の書き方

```c
#define SWAP(type, a, b)  \
  {                       \
    type __tmp__ = a;     \
    a = b;                \
    b = __tmp__;          \
  }
```


## SWAPマクロ 基本形 (2-1)

do ~ while (0) を付ける

```c
#define SWAP(type, a, b)  \
  do {                    \
    type __tmp__ = a;     \
    a = b;                \
    b = __tmp__;          \
  } while (0)
```


## SWAPマクロ 基本形 (2-2)

- do ~ while (0) を付ける利点
  - セミコロンの付加を強制できる
    - 読み手を混乱させない
    - エディタを混乱させない
  - 以下の中括弧省略型if文でエラーにならない

```c
if (a)
  SWAP(int a, b);
else
  printf("Hello\n");
```


## SWAPマクロ 基本形 (3)

- C言語に参照型はない
  - 引数に破壊的変更を加える場合は，ポインタで渡したい

```c
#define SWAP(type, a, b)  \
  do {                    \
    type __tmp__ = *(a);  \
    *(a) = *(b);          \
    *(b) = __tmp__;       \
  } while (0)
```


## SWAPマクロ 応用形 (4-1)

- 引数に型を取るのは不恰好
  - ポインタを渡すだけで，値を交換して欲しい
- xorを用いて，値を交換する
- 変数を宣言しないので，カンマ演算子でつないで式にできる

```c
#define SWAP(a, b)  \
  (                 \
    *(a) ^= *(b),   \
    *(b) ^= *(a),   \
    *(a) ^= *(b)    \
  )
```

- カッコいい書き方は以下の通り
  - 右から評価するか左から評価するかは未定義だが，大抵のコンパイラならうまくいく

```c
#define SWAP(a, b)  (*(a) ^= *(b) ^= *(a) ^= *(b))
```


## SWAPマクロ 応用形 (4-2)

- xorを用いたSWAPマクロの欠点
  - 同じアドレスの変数に用いたとき，変数の値が0になる
    - ```SWAP(a, a)```など
    - 同じアドレスを指してないか確認することで**解決可能**
  - 整数のポインタにしか適用できない
    - キャストした場合は，この限りではない


## SWAPマクロ 応用形 (4-3)

- 同じアドレスかどうかをチェックする
  - 条件演算子

```c
// 式全体の値はa == b のとき，0
// a != b のとき1
#define SWAP(a, b)     \
  (((a) == (b)) ? 0 :  \
   (                   \
     *(a) ^= *(b),     \
     *(b) ^= *(a),     \
     *(a) ^= *(b),     \
     1                 \
  ))
```


## SWAPマクロ 応用形 (5-1)

- xorでは浮動小数点に対応できなかった
  - 加減算によるSWAPで対応

```c
#define SWAP(a, b)       \
  (                      \
    *(a) += *(b),        \
    *(b) = *(a) - *(b),  \
    *(a) -= *(b)         \
  )
```

- カッコいい書き方は以下の通り
  - 右から評価するか左から評価するかは未定義だが，大抵のコンパイラならうまくいく

```c
#define SWAP(a, b)  (*(a) += *(b) -= *(a) = *(b) - *(a))
```


## SWAPマクロ 応用形 (5-2)

- 加減算を用いたSWAPマクロの欠点
  - 同じアドレスの変数に用いたとき，変数の値が0になる
    - ```SWAP(a, a)```など
    - 同じアドレスを指してないか確認することで**解決可能**
  - 桁落ちが発生する可能性がある
    - ```a```が巨大な値で```b```が小さな値のときなど
  - 整数型と浮動小数点型のポインタにしか適応できない
    - 構造体は不可


## SWAPマクロ 応用形 (5-3)

- 同じアドレスかどうかをチェックする
  - 条件演算子

```c
// 式全体の値はa == b のとき，0
// a != b のとき1
#define SWAP(a, b)        \
  (((a) == (b)) ? 0 :     \
   (                      \
     *(a) += *(b),        \
     *(b) = *(a) - *(b),  \
     *(a) -= *(b),        \
     1                    \
   ))
```


## SWAPマクロ 完全形 (6-1)

- GNU拡張文法を用いる
  - ```typeof```演算子で，引数の型を取得する
  - 文を式にする

```c
#define SWAP(a, b)               \
  ({                             \
   typeof(*(a)) __tmp__ = *(a);  \
   *(a) = *(b);                  \
   *(b) = __tmp__;               \
  })
```


## SWAPマクロ 完全形 (6-2)

- C++11では，GNU拡張文法を用いなくても，完璧なSWAPマクロを実現できる
  - ```typeof```の代わり
    - ```decltype```演算子
  - 動的型付け
    - ```auto```

```c
#if defined(__cplusplus) && __cplusplus >= 201103L
#  define SWAP(a, b)                     \
    {                                    \
      // decltype(*(a)) __tmp__ = *(a);  \
      auto __tmp__ = *(a);               \
      *(a) = *(b);                       \
      *(b) = __tmp__;                    \
    }
#elif defined(__GNUC__)
#  define SWAP(a, b)                \
    ({                              \
      typeof(*(a)) __tmp__ = *(a);  \
      *(a) = *(b);                  \
      *(b) = __tmp__;               \
    })
#endif
```


## 競技プログラミングでよくあるループマクロ (1)

- 競技プログラミングでは，ループもマクロ化する
  - タイプ数を少しでも減らすため

```cpp
#define FOR(i, a, b)  for (int i = a; i < b; i++)
#define REP(i, n)     for (int i = 0; i < n; i++)
#define LOOP(n)       for (int __loop_tmp_var__ = 0; i < n; i++)
```


## 競技プログラミングでよくあるループマクロ (2)

- 使用例

```cpp
FOR(i, 10, 20) {
  printf("%d\n", i);
}

REP(i, n) {
  array[i] = n;
}

LOOP(10) {
  puts("Hello World!");
}
```


## C言語でラムダ (1)

- GNU拡張文法を用いれば，C言語でもラムダが実現できる
  - 複文の式化
  - 関数内関数定義（C言語のみのGNU拡張文法）

```c
#define LAMBDA(RETTYPE, ARGLIST, BODY)            \
  ({                                              \
    RETTYPE __lambda_funcion__ ARGLIST { BODY; }  \
    __lambda_funcion__;                           \
  })
```


## C言語でラムダ (2)

使用例

```c
int main(void) {
  int i;
  int array[5] = {4, 1, 7, -2, 3};
  int len;

  // 関数ポインタに無名関数を渡す例
  int (*sum)(int, int, int) = LAMBDA(int, (int a, int b, int c), {
    return a + b + c;
  });
  // 関数ポインタに格納した無名関数を呼び出す
  printf("2 + 3 + 5 = %d\n", sum(2, 3, 5));
  // その場で無名関数を呼び出す例
  printf("6 - 1 = %d\n", LAMBDA(int, (int a, int b), {
    return a - b;
  })(6, 1));
  // qsort()関数の第4引数に渡す例
  qsort(array, (sizeof(array) / sizeof(array[0])), sizeof(array[0]),
    LAMBDA(int, (const void *a, const void *b), {
      return *(int *)a - *(int *)b;
    })
  );
  for (i = 0; i < (sizeof(array) / sizeof(array[0])); i++) {
    printf("array[%d] = %d\n", i, array[i]);
  }
  return 0;
}
```




## 参考文献

- [コラム:SWAPマクロの完成形](http://homepage3.nifty.com/mmgames/c_guide/c_swap.html)
- [CプリプロセッサでBrainfuck](http://www.kotha.net/bfi/index_ja.html)
- [Microsoft Visual C++ _MSC_VERの値メモ - けんちゃんのブログ](http://mikacolove.blog.fc2.com/blog-entry-6.html)
- [灘高校パソコン研究部 2012年度部誌 華麗なるC++テクニック!](http://www.npca.jp/works/magazine/2012_5/)
- [Template Meta Programming入門から応用まで](http://www.slideshare.net/yoshihikoozaki5/tmp-web-28084468)
- [difference between gcc and vc](http://homepage1.nifty.com/herumi/prog/gcc-and-vc.html)
- [C++14 時代の constexpr プログラミング作法 - ボレロ村上 - ENiyGmaA Code](http://boleros.hateblo.jp/entry/20131203/1386011596)
- [本の虫: 最新のconstexpr](http://cpplover.blogspot.jp/2013/04/constexpr.html)
- [Boost.MPLでBrainf*ckのプログラムをコンパイル時に動かしてみた - ぬいぐるみライフ(仮)](http://d.hatena.ne.jp/mickey24/20100909/boost_mpl_de_brainfuck)
