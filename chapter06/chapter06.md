## 第6回
<h1>プログラミング勉強会</h1>

2014年7月10日
担当: koturn




## 目次

- C言語の規格
  - C99
  - C11
- C++の規格
  - C++11
  - C++14




# C99




## C99とは

- ISOで定められたC言語の規格
- C++からの昨日の逆輸入
- 各コンパイラが独自実装していた機能の導入


## 対応コンパイラ

- gcc4.6以上より実装開始
  - `-std=gnu11` を付加してコンパイル
    - 特別，GNU拡張を利用しない理由がないならこれにする
  - `-std=c11` を付加してコンパイル
    - GNU拡張を利用できない
- clang3.0以上より実装開始


## C11がコンパイル時に対応しているかどうか

- `__STDC_VERSION__` マクロを調べる
  - `__STDC_VERSION__ >= 199901L` なら，C99対応の環境
  - `__STDC_VERSION__ <  199901L` なら，C99非対応の環境

```c
#if defined(__STDC_VERSION__) && __STDC_VERSION__ >= 199901L

// C99用のソースコードの記述

#else

// C99以前用のソースコードの記述

#endif
```




## 変数宣言位置の緩和

- 変数の宣言位置がブロックの先頭以外でも可能になった
  - gccでは独自拡張で導入されていたもの

```c
int main(void) {
  int a = 10;  // 従来のC言語でもOK
  printf("a = %d\n", a);
  int b = 20;  // C99で許可された
  printf("b = %d\n", b);

  // for文内での変数宣言も可能に
  for (int i = 0; i < 10; i++) {
    puts("Hello World");
  }
}
```




## inline (1)

- C++からの逆輸入
- インライン関数を定義する


## inline (2)

- C99以前も各コンパイラで独自実装されていた
- gcc, clang
  - そのまま `inline` キーワードを使用可能
- MSVC
  - `__inline`


## inline (3)

- 以下のようなマクロを記述すると，C99でなくとも `inline`
  が使用可能になるかもしれない
- 対応していない環境では `inline` を消す

```c
#ifndef __cplusplus
#  if defined(_MSC_VER)
#    define inline      __inline
#    define __inline__  __inline
#  elif !defined(__GNUC__) || !define(__STDC_VERSION__) || __STDC_VERSION__ < 199901L
#    define inline
#    define __inline
#  endif
#endif
```


## inline (4)

- 各コンパイラの独自拡張で，強制的なインライン展開の指定が可能
- gcc
  - gcc 4.0 以降
  - 常にインライン展開することを指示
  - `__attribute__((always_inline)) inline`
- MSVC
  - 可能な限りインライン展開することを指示
  - `__forceinline`


## inline (5)

- 各コンパイラの独自拡張で，インライン展開しないことも指示できる

- gcc
  - `__attribute__((noinline))`
- MSVC
  - `__declspec(noinline)`




## restrict (1)

- ポインタ変数の最適化指示
- そのポインタが指す領域を他のポインタが指さないことをコンパイラに伝達する
  - コンパイラの自動ベクトル化（SIMD化）の判断の手助けになる


## restrict (2)

- C99以前も各コンパイラで独自実装されていた
  - `restrict` はC++14にもないが，独自実装しているコンパイラなら使用可能
- gcc, clang
  - `__restrict`
  - `__restrict___`
- MSVC
  - `__restrict`


## restrict (3)

- 以下のようなマクロを記述すると，C99でなくとも `restrict`
  が使用可能になるかもしれない
- 対応していない環境では `restrict` を消す

```c
#if _MSC_VER >= 1400
#  define restrict      __restrict
#  define __restrict__  __restrict
#elif __cplusplus
#  define restrict      __restrict
#elif !define(__STDC_VERSION__) || __STDC_VERSION__ < 199901L
#  if defined(__GNUC__)
#    define restrict    __restrict
#  else
#    define restrict
#    define __restrict
#    define __restrict__
#  endif
#endif
```


## restrict 例(1)

- restrict無しの実装

```c
void vec_add(
  float *z,
  const float *x,
  const float *y,
  size_t N)
{
  // zとx，zとyが指す領域がオーバーラップしている可能性もあるので
  // 自動ベクトル化すべきかどうか判断できない
  for (i = 0; i < N; i++) {
    z[i] = x[i] + y[i];
  }
}
```


## restrict 例(2)

- restrict付きの実装

```c
void vec_add(
  float *restrict z,
  const float *restrict x,
  const float *restrict y,
  size_t N)
{
  for (i = 0; i < N; i++) {
    z[i] = x[i] + y[i];
  }
  // 以下の擬似的なコードと等価
  /**
  for (i = 0; i < N; i += 4) {
    z[i: i + 3] = x[i: i + 3] + y[i: i + 3];
  }
  **/
  もしくは，
  for (i = 0; i < N; i += 4) {
    // 以下が同時に行われる
    z[i]     = x[i]     + y[i];
    z[i + 1] = x[i + 1] + y[i + 1];
    z[i + 2] = x[i + 2] + y[i + 2];
    z[i + 3] = x[i + 3] + y[i + 3];
  }
}
```




## 新しいヘッダ

- いくつかのヘッダが新しく導入された
  - `<inttypes.h>`
  - `<stdbool.h>`
  - `<stdint.h>`




## 1行コメントのサポート

- `//` で始まる1行コメントが正式に導入された




## 要素数が変数である静的配列

- 要素数に変数をとることが可能となった
  - 配列の記法で動的配列を実現
- メモリ解放を記述する必要はない
- `sizeof` 演算子は実行時に処理される

```c
int func(int n) {
  char buf[n];
  return sizeof(buf);
}
```




## 可変引数マクロ

- C99から可変引数マクロが許容された

```c
#define dbg_print(fmt, ...)  fprintf(stderr, fmt, __VA_ARGS__)

int a = 10;
dbg_printf("a = %d\n", a);
// => fprintf(stderr, "a = %d\n", a) と展開
```




# C11




## C11

- 2014年現在，最新のC言語の規格
- モダンな機能が多数取り入れられた
  - C++11と同様のものもある


## 対応コンパイラ

- gcc4.6以上より実装開始
  - `-std=gnu11` を付加してコンパイル
    - 特別，GNU拡張を利用しない理由がないならこれにする
  - `-std=c11` を付加してコンパイル
    - GNU拡張を利用できない
- clang3.0以上より実装開始


## C11がコンパイル時に対応しているかどうかをチェック

- `__STDC_VERSION__` マクロを調べる
  - `__STDC_VERSION__ >= 201112L` なら，C11対応の環境
  - `__STDC_VERSION__ <  201112L` なら，C11非対応の環境

```c
#if defined(__STDC_VERSION__) && __STDC_VERSION__ >= 201112L

// C11用のソースコードの記述

#else

// C11以前用のソースコードの記述

#endif
```




## _Genericキーワード

- C11における疑似的なオーバーロード機能の提供

- 従来のC言語ではオーバーロードできない
  - 引数の型と名前が異なる多数の関数を呼び出す必要があった
- `_Generic(foo, TYPE1: IDT1, TYPE2: IDT2, ..., TYPEN: IDTN)` という形式
  - 変数 `foo` の型によって，`_Generic` 全体が `IDT*` に置き換わる
  - コンパイル時の型による条件分岐が可能に


## _Genericの例

```c
#include <stdio.h>
#include <stdlib.h>

#define myabs(x)  \
  _Generic((x), int: myabs_i, \
                double: myabs_f,  \
                default: myabs)(x)

int myabs_i(int x) {
  return x > 0 ? x : -x;
}

double myabs_f(double x) {
  return x > 0 ? x : -x;
}

int main(int argc, char *argv[]) {
  myabs(2);    // => myabs_i(2) と等価
  myabs(2.2);  // => myabs_f(2.2) と等価
  return EXIT_SUCCESS;
}
```




## _Noreturn

- その関数から呼出元へ制御が戻らないことを明示
  - 戻り先のアドレスをスタックに積む必要がなくなる
  - 例:
    - `exit()` 関数
- C++11における `[[noreturn]]` と同じ
- 対応コンパイラ
  - gcc4.8以上
  - clang3.3以上




## _Static_assert

- コンパイル時のassert
- C++11における `static_assert` と同じ
- 形式は `_Static_assert(expr, msg)`

```c
#define N  100

int main(void) {
  _Static_assert(N <  1000, "test1");  // pass
  _Static_assert(N != 1000, "test2");  // fail
  return 0;
}
```




## アラインメント関係の機能追加

- `_Alignas` 指定子
- `alignof` 演算子
- `aligned_alloc()` 関数
- `<stdalign.h>` ヘッダ


## _Alignas演算子

- 宣言する変数のアラインメントを指定する
- C++11における `alignas` と同じ

```c
_Alignas(16) int a = 10;
```


## alignof演算子

- 指定した変数のアラインメントを取得する
  - C++11の `alignof` 演算子と同じ

```c
int a = 10;
_Alignas(32) int a = 30;
printf("alignof(a) = %d", alignof(a));
// => alignof(a) = 4 (処理系による)
printf("alignof(b) = %d", alignof(b));
// => alignof(b) = 32
```




## gets()関数の廃止

- C11から`gets()` 関数を廃止した
  - `gets()` 関数はバッファオーバーフローの危険性があるため
- `gets()` 関数を用いたプログラムはコンパイルエラーとなる
  - 過去のプログラムがコンパイルできない可能性
- 代わりにセキュアな`gets_s()` 関数が実装
  - しかし，実装は必須ではない
  - 境界チェックを行う
  - MSVCでは独自拡張で導入されていた




# C++11




## C++11とは

- モダンな言語機能を取り入れたC++の規格
- g++/clangでのみ利用可能
  - MSVCは中途半端にサポートしている
- 実装途中ではC++0xと呼ばれていた
- 詳しくは [N3337](http://boleros.x0.com/data/n3337.pdf) というドラフトを参照


## C++11がコンパイル時に対応しているかどうかをチェック

- `__cplusplus` マクロを調べる
  - `__cplusplus >= 201103L` なら，C++11対応の環境
  - `__cplusplus <  201103L` なら，C++11非対応の環境
- `__GXX_EXPERIMENTAL_CXX0X__` が定義されているかどうか
  - g++のみ
  - C++0xでの実装

```c
#if defined(__cplusplus) && __cplusplus >= 201103L

// C++11用のソースコードの記述

#else

// C++11以前用のソースコードの記述

#endif

```


## g++/clangでのC++11のコンパイル

- g++

```sh
$ g++ -std=gnu++11 -c foo.cpp -o foo.o
$ g++ foo.o -o foo.out
```

- clang

```sh
$ clang -std=gnu++11 -c foo.cpp -o foo.o
$ clang foo.o -o foo.out
```




## autoによる型推論

- 型推論のためのキーワード
  - C\#における `var` のようなもの
- 宣言と同時に初期化する変数にのみ使用可能

```cpp
auto foo = bar();
```


## decltype

- コンパイル時に型を取得することが可能

```cpp
int x - 10, y = 20;
decltype(x + y) z = x + y;
```




## nullptr

- C++では `#define NULL 0` と定義されていた
  - ポインタではなく，整数として扱われるという問題

```cpp
void foo(char *);
void foo(int);

...

foo(NULL);  // void foo(int); の呼び出しになってしまう
foo(nullptr);  // void foo(char *); の呼び出し
```




## 範囲for

- 他の言語における `foreach` 文にあたる
- リストの要素を順に走査する処理の記述が可能

```cpp
int my_array[5] = {1, 2, 3, 4, 5};
for (int& x : my_array) {
  x *= 2;
}
```




## ラムダ関数 (1)

- 他言語でよくみられるラムダが使用可能になった
- クロージャも可能

```cpp
// 例1
// [=]  外部変数のキャプチャ: =はコピー，&なら参照
// (int x, int y)  引数
// int  返り値の型
// { ... }  関数の本体
[=](int x, int y) -> int { int z = x + y; return z + x; }
```


## ラムダ関数 (2)

```cpp
// 例2: 省略系
// []  この場合，キャプチャはコピーで行われる
// (int x, int y)  引数
// 返り値は省略でき，この場合その型はdecltype(x + y)
// { ... }  関数の本体
[](int x, int y) { return x + y; }
```


## ラムダの例

```cpp
// 配列を絶対値順にソート
int array[] = {9, 7, 5, 3, 1, -2, -4, -6, -8, 0};
std::sort(array, array + 10, [](int x, int y) {
  return std::abs(x) < std::abs(y);
});

// 配列の各値を表示
std::for_each(array, array + 10, [](int x) {
  std::cout << x << " ";
});
```




## 新しい関数宣言

- 従来のテンプレートでは対応できないパターンのために新しい関数宣言が提案された
- 従来のものとは見た目がかなり異なる


## 従来の関数宣言の問題点

- 以下のコードを例にとる
  - 加算演算子により `add` を実現する関数
  - 戻り値の型は `x` と `y` の型によって決定される
- コンパイルエラーとなる
  - `decltype` のパース時点で， `x` と `y` は登場していないため
  - C++コンパイラは基本的に1pass

```cpp
template<typename T1, typename T2>
decltype(x + y) myadd(const T1& x, const T2& y) {
  return x + y;
}
```


## C++11の関数宣言の例

- 前述の問題を解決するために，以下の関数宣言が提案された
  - 戻り値の型を後置することで，引数の型をとることが可能
    - `decltype` が活用可能
  - ここでの `auto` は変数宣言の `auto` とは異なる意味を持つ

```cpp
template<typename T1, typename T2>
auto add(const T1& x, const T2& y) -> decltype(x + y) {
  return x + y;
}
```




## override

- 明示的な関数のオーバーライド
- typeにより，overrideし損ねたときにコンパイルエラーを出す
  - Javaの `@Override` と同じ

```cpp
class SuperClass {
public:
  virtual void func(float);
};

class SubClass : SuperClass {
public:
  // func を fnuc とtypo
  // コンパイルエラーとなる
  virtual void fnuc(int) override;
};
```


## final

- 仮想関数のオーバーライドの禁止

```cpp
class SuperClass {
public:
  virtual void func() final;
};

class SubClass : SuperClass {
public:
  void func();  // エラー
};
```




## noexcept

- 関数が例外を投げないことを示す
  - 最適化につながる

```c
void myabs(int a, int b) noexcept {
  return a > b ? a : b;
}
```


## C++11以前でのnoexcept

- 各コンパイラで例外を投げないことを明示する独自拡張があった
  - gccなら `__attribute__((nothrow))`
  - MSVCなら `__declspec(nothrow)`
- C++11のnoexceptと宣言位置が異なる点が問題
  - マクロで互換性の問題を解決できない

```c
#if defined(__GNUC__)
__attribute__((nothrow))
int myabs(int a, int b)  {
  return a > b ? a : b;
}

#elif defined(_MSC_VER)
__declspec(nothrow)
int myabs(int a, int b) noexcept {
  return a > b ? a : b;
}
#endif
```




## usingによる別名テンプレート (1)

- C++11以前では， `typedef` のテンプレートを作ることができなかった

```cpp
template <typename T1, typename T2>
class MyClass;

template <typename T2>
typedef MyClass<int, T2>  FOO;
```


## usingによる別名テンプレート (2)

- C++11では `using` というキーワードにより，別名のテンプレートを作ることが可能となった
  - `using namespace foo;` の `using` と意味は異なる

```cpp
template <typename T1, typename T2>
class MyClass;

template <typename T2>
using FOO = MyClass<int, T2>;
```


## usingによる型のalias

- 従来は `typedef` で型の別名を定義していた
- C++11でusingを用いた型の別名定義が可能となった

```cpp
typedef int32  int;  // 従来の型の別名定義
using int32 = int;   // 新しい型の別名定義
```




## constexpr

- コンパイル時処理のための機構
- 従来はテンプレートで行っていた処理を普通の関数と同じ形式で記述可能となった
  - 従来以上の機能を持つ
- 変数と関数に付与可能


## constexpr関数の制限

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


## constexpr 例(1)

- 従来のテンプレートによるフィボナッチ数列のコンパイル時計算

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


## constexpr 例(2)

- `constexpr` によるフィボナッチ数列のコンパイル時計算

```cpp
constexpr unsigned int
fibonacci(unsigned int n)
{
  return n > 0 ? fibonacci(n - 1) + fibonacci(n - 2)
    : n == 1 ? 1
    : 0;
}
```




## 関数の属性記述

- 関数に `[[...]]` という形式で属性を記述できるようになった
  - gccの `__attribute__(...)` に相当
  - MSVCの `__declspec(...)` に相当


## noreturn属性

- その関数から制御が戻らないことを示す
  - C11における `_Noreturn` と同様
- 関数に `[[noreturn]]` を付加して記述

```cpp
[[noreturn]] void print_exit(const char *msg) {
  std::cout << msg << std::endl;
  std::exit(1);
}
```


## noreturnのラッパーマクロ

```cpp
#if (__cplusplus >= 201103L || defined(__GXX_EXPERIMENTAL_CXX0X__)) &&
      (GNUC_PREREQ(4, 8) || CLANG_PREREQ(3, 3))
#  define ATTR_NORETURN  [[noreturn]]
#elif defined(_MSC_VER) && _MSC_VER >= 1300
#  define ATTR_NORETURN  __declspec(noreturn)
#elif defined(__GNUC__)
#  define ATTR_NORETURN  __attribute__((noreturn))
#elif defined(__STDC_VERSION__) && __STDC_VERSION__ >= 201102L
#  define ATTR_NORETURN  _Noreturn
#else
#  define ATTR_NORETURN
#endif
```


## carries_dependency属性

- 関数の間で，データの依存性を伝播するために使用
- 詳細は割愛する




## static_assert

- コンパイル時のassert
- `constexpr` などと併用する

```cpp
constexpr int N = 10;
static_assert(N + 10 == 20, "test1");  // pass
static_assert(N + 10 != 20, "test2");  // fail
```


## C++11以前でのstatic_assertの実装

- 静的配列を利用して， `static_assert` を実装していた
  - エラー時のメッセージを指定できない

```cpp
#define my_static_assert(expr)  \
  typedef char __STATIC_ASSERT_ARRAY__[(expr) ? 1 : -1]

int N = 10;
my_static_assert(N == 10);  // pass
my_static_assert(N != 10);  // fail
```




## 右辺値参照とmoveセマンティクス

- コピーコンストラクタの呼び出し回数を減らすための機構
- 所有権の委譲


## 従来の問題点 (1)

- 以下のコードの問題点
  1. 関数内で一時的に `std::vector` である `v` を生成
  2. 生成した `v` に要素を追加
  3. 関数の呼び出し元に，`v` をコピーして返却
  4. 呼び出し元でコピーされた `v` を受け取る

```cpp
std::vector<int> func(void) {
  std::vector<int> v;
  for (int i = 0; i < 10; i++) {
    v.push_back(i);
  }
  return v;
}

int main(void) {
  std::vector<int> v = func();

  return 0;
}
```


## 従来の問題点 (2)

- 従来では，以下のように前述の問題を解決
- しかし，破壊的変更を必要とし，シームレスではない

```cpp
void func(std::vector<int> v) {
  for (int i = 0; i < 10; i++) {
    v.push_back(i);
  }
}

int main(void) {
  std::vector<int> v;
  func(v);

  return 0;
}
```


## moveを用いた解決

- `std::move()` 関数は前述の問題を解決
  - 返り値の返却にコピーコンストラクタの呼び出しをしない
  - シームレスな関数の設計が可能

```cpp
std::vector<int> func(void) {
  std::vector<int> v;
  for (int i = 0; i < 10; i++) {
    v.push_back(i);
  }
  return std::move(v);
}

int main(void) {
  std::vector<int> v = func();

  return 0;
}
```




# C++14




## C++14とは

- C++11のマイナーバージョンアップ
- C++11と比較して，大きな変更はあまりない
- 詳しくは[N3936](http://boleros.x0.com/data/n3936.pdf)というドラフトを参照


## g++/clangでのC++14のコンパイル

- g++

```sh
$ g++ -std=gnu++1y -c foo.cpp -o foo.o
$ g++ foo.o -o foo.out
```

- clang

```sh
$ clang -std=gnu++1y -c foo.cpp -o foo.o
$ clang foo.o -o foo.out
```




## constexprの制限緩和

- 関数内で変数宣言可能
- if文とswitch文の許可
- 全てのループ文の許可(for，範囲for，while，do-while)
- 変数書き換えの許可
  - 非constexpr変数は不可
  - グローバル変数は不可
- gccは現在(ver. 4.10 現在)非対応
- clangは(ver3.3以降)対応している
- MSVCは非対応




## 関数属性の追加

- 関数属性が追加された
- `deprecated` 属性
  - 廃止予定の関数であることを示す
  - gccの `__attribute__((deprecated))` と同じ


## 導入されるかもしれない属性

- ```pure```属性
  - 純粋関数であることを示す
  - gccの `__attribute__((const))` と同じ




## 2進数リテラル

- 2進数で考えう必要のある場面で，わざわざ16進数にする必要がなくなった

```cpp
int a = 0b00001101;
```



## 数値区切り

- 数値を区切ることにより，大きな数をみやすくすることが可能になった

```cpp
int a = 10'000'000;
```




## 実行時配列サイズ指定

- 実行時の数で配列サイズを指定することが可能になった
- C99のものとの違い
  - `sizeof()` はサポートされない

```cpp
void func(int a) {
  char buf[a];
}
```




## 参考文献

- [プログラミング言語 C の新機能](http://seclan.dll.jp/c99d/)
- [N3337](http://boleros.x0.com/data/n3337.pdf)
- [右辺値参照とムーブ・セマンティクス | S.F.Page](http://www.enoie.net/blog/2013/03/%E5%8F%B3%E8%BE%BA%E5%80%A4%E5%8F%82%E7%85%A7%E3%81%A8%E3%83%A0%E3%83%BC%E3%83%96%E3%83%BB%E3%82%BB%E3%83%9E%E3%83%B3%E3%83%86%E3%82%A3%E3%82%AF%E3%82%B9/)
- [N3936](http://boleros.x0.com/data/n3936.pdf)
- [C++14の新機能](http://ezoeryou.github.io/kabukiza-tech2-slide/)
