## 第2回
<h1>プログラミング勉強会</h1>

2014年5月22日
担当: koturn




## 目次

- C/C++のマナー
  - static
  - const
  - エラーメッセージ
  - メモリ確保のエラー
  - C++のキャスト
  - 変数の宣言
  - ヘッダファイル
  - main()関数の書き方




## static

- staticは場所により意味が異なる
  - グローバル領域
  - 関数内
  - クラス内


## グローバル領域でのstatic

- そのソースコードでしか参照できないようにする
  - コンパイル時にシンボル名を変更する
  - 他のソースコードで同じ名前の変数，関数を使用できる


## 関数内でのstatic

- 変数の寿命を永続化
- 関数をシングルトンなセカンドクラスオブジェクトとみなせば，クロージャの機能を
  果たすといえる


## クラス内でのstatic

- Javaと同じ
- クラスに所属することを示す




## const

- 値が変更できないことを表す
  - Javaのfinalより強力
- ポインタでは，つける場所により意味が異なる
- 関数にポインタを渡すときに重要


## ポインタのconst

- const int \*ptr = &value;
  - 値が変更できないことを示す
  - アドレスは変更可能
- int \*const ptr = &value;
  - アドレスが変更できないことを示す
  - 値は変更可能


## 関数にポインタを渡すとき (1)

- 関数がポインタに引数をとるのは，以下の3つの場面が考えられる
  - 配列の要素をなぞるとき
  - データサイズの大きい構造体を渡すとき
  - 引数を変更したいときや複数の返り値を持ちたいとき
- 1番目，2番目は引数に変更を加えない
- 3番目は引数自体に変更を加える(破壊的変更)


## 関数にポインタを渡すとき (2)

- constを付けることで，変更されないことを明示
  - 第一に可読性のため
  - 副産物として，バグを防ぐ効果

```c
char *mystrcpy(char *dst, const char *src) {
  while ((*dst++ = *src++) != '\0');
  return dst;
}
```




## エラーメッセージは標準エラーに

- 標準出力は普通のメッセージ表示に
- 標準エラー出力はエラーメッセージの表示に

```c
static const char *FILENAME = "test.txt";
FILE *fp = fopen(FILENAME, "r");
if (fp == NULL) {
  fprintf(stderr, "Unable to open %s\n", FILENAME);
  exit(EXIT_FAILURE);
}
```


## リダイレクト (1)

以下のプログラムを例にとる

- sample.c

```c
#include <stdio.h>

int main(void) {
  printf("stdout: Hello World!\n");
  fprintf(stderr, "stderr: Hello World!\n");
  return 0;
}
```


## リダイレクト (2)

- 普通にコンパイル&実行

```sh
$ gcc sample.c -o sample.out
$ ./sample.out
stdout: Hwllo World!
stderr: Hwllo World!
```


## 標準出力のリダイレクト

- 以下のようにして，標準出力をリダイレクト
  - 標準エラー出力はリダイレクトされない

```sh
$ ./sample.out > sample-stdout.log
stderr: Hello World!
```

- 中身を確認

```sh
$ cat ./sample-stdout.log
stdout: Hello World!
```


## 標準エラー出力のリダイレクト

- 以下のようにして，標準エラー出力をリダイレクト
  - 標準出力はリダイレクトされない

```sh
$ ./sample.out 2> sample-stderr.log
stdout: Hello World!
```

- 中身を確認

```sh
$ cat ./sample-stderr.log
stderr: Hello World!
```


## 標準出力，標準エラー出力を2つともリダイレクト

- 以下のようにする
  - ややこしいので，覚えにくい...
    - 2>&1 ?
    - 2&>1 ?
    - 1>&2 ?
    - 1&>2 ?

```sh
$ ./sample.out > sample.log 2>&1
```

- 中身を確認

```sh
$ cat ./sample.log
stdout: Hello World!
stderr: Hello World!
```


## コンソールに表示しつつリダイレクト

- teeコマンドを用いる

```sh
$ ./sample.out 2>&1 | tee sample.log
stdout: Hello World!
stderr: Hello World!

$ cat ./sample.log
stdout: Hello World!
stderr: Hello World!
```




## メモリ確保エラーのチェック

- 動的なメモリ確保は失敗することがある
- C言語では以下のようにエラーチェック
  - `malloc()` 等の返り値がNULLかどうか

```c
int *array = calloc(10, sizeof(int));
if (array == NULL) {
  fprintf(stderr, "Unable to allocate memory\n");
  exit(EXIT_FAILURE);
}
```

- C++の `new` と `delete` の場合
  - `std::bad_alloc` 例外の確認

```cpp
try {
  int *array = new int[10];
} catch (std::bad_alloc &e) {
  std::cerr << "Unable to allocate memory" << std::endl;
  std::exit(EXIT_FAILURE);
}
```


## メモリ解放

- メモリを解放した後は，そのポインタをNULLクリアするのがよい
  - 間違って再利用するのを防ぐため

C言語の場合

```c
free(array);
array = NULL;
```

C++の場合

```cpp
delete array;
array = NULL;
```


## ファイルオープンのエラーチェック

- メモリ確保とほぼ同様

C言語の場合

```c
FILE *fp = fopen("sample.txt", "r");
if (fp == NULL) {
  fprintf(stderr, "Unable to open file: %s\n", sample.txt);
  exit(EXIT_FAILURE);
}
```

C++の場合

```cpp
std::ifstream ifs("sample.txt");
if (ifs.fail()) {
  std::cerr << "Unable to allocate memory" << std::endl;
  std::exit(EXIT_FAILURE);
}
```


## ファイルのクローズも忘れない

- 用が済んだファイルはクローズする

C言語の場合

```c
fclose(fp);
fp == NULL;
```

C++の場合

```cpp
// デストラクタでクローズされるので，呼び出さなくてもよい
ifs.close();
```




## C言語の変数の宣言位置 (1)

- C言語の文法では，変数は**ブロックの先頭**で宣言しなければならない
  - *関数の先頭*ではない
- 以下の場所では変数を宣言できない
  - 文の先頭以外
  - for文の中
- gccでは拡張機能により，ブロックの先頭以外でも変数を宣言できた
  - for文の中ではできない


## C言語の変数の宣言位置 (2)

具体例を示す

```c
int main(void) {
  int a = 10;  // ブロックの先頭
  printf("a = %d\n", a);
  int b = 20;  // NG: 先頭以外(gccならOK)
  printf("b = %d\n", b);
  if (a == 10) {
    int c = 30;  // OK: ブロックの先頭
    printf("a * %d = %d\n", c, a * c);
  }

  // ブロックを作成
  {
    int d = 5;  // OK: これもブロックの先頭
    printf("a / %d = %d\n", d, a / d);
  }

  // NG: for文の中では宣言できない
  for (int i = 0; i < 10; i++) {
    puts("Hello!");
  }
}
```


## C99規格での変数の宣言位置

- C99規格から，C++の変数の宣言位置を許容するようになった(逆輸入)
- 以下の場所で変数を宣言することが可能になった
  - ブロックの途中
  - for文の中
- 以下のコードはC99ならOK

```c
int main(void) {
  int a = 10;
  printf("a = %d\n", a);
  int b = 20;
  printf("b = %d\n", b);
  for (int i = 0; i < 10; i++) {
    puts("Hello!");
  }
}
```




## ヘッダファイルの書き方

- ヘッダファイルには以下のものを書く
  - extern宣言の変数
  - 関数プロトタイプ
  - マクロ定義
  - 型定義(typedef)
  - 列挙体定義(enum)
  - クラス宣言
  - インライン関数
  - テンプレート関数
- 実体は書かない！


## インクルードガード

- 2重インクルードを防ぐために必要
- 2つの方法がある
  - \#define を用いた手法
  - \#pragma を用いた手法


## #define によるインクルードガード

- 以下のような記述をする

```c
#ifndef FOO_H
#define FOO_H

... (ヘッダファイルの内容) ...

#endif
```


## #pragma によるインクルードガード

- 最近のコンパイラだと対応している

```c
#pragma once

... (ヘッダファイルの内容) ...
```




## main()関数の書き方 (1)

- 返り値の型はint

```c
// NG: 返り値の型を省略している
main(void) {

}

// NG: void型の返り値
void main(void) {

}

// OK: 返り値の型がint型
int main(void) {

}
```


## main()関数の書き方 (2)

- 引数は，以下のどちらか
  1. `void`
  2. `int argc, char *argv[]`

```c
// NG: 引数がvoidでない(C++ならOK)
int main() {

}
// OK: 引数がvoid
int main(void) {

}
// OK: int argc, char *argv[]
int main(int argc, char *argv[]) {

}
// NG: MSVCの環境変数参照のための拡張
int main(int argc, char *argv[], char *env[]) {

}
```




## 目次 (2)

- CからC++への移行
  - using namespace について
  - C言語のヘッダと関数
  - 初期化リスト
  - メソッドの実装
  - キャスト
  - extern "C"について


## using namespace について

- むやみやたらに用いない
  - どのライブラリの関数か，一目でわからなくなるため
- 用いるとしても，グローバルスコープで用いない
  - ヘッダファイルのグローバルスコープで使用するのは最悪


## C++でC言語の関数を利用するには(1)

- C言語の関数を仕様するにあたり，<stdio.h>などのインクルードは禁止
- 名称が変更されたヘッダを用いるべき
  - 拡張子.hが除かれ，先頭にcを追加したもの
- 以下は対応表の一部

C言語ヘッダ | C++ヘッダ
------------|----------
math.h      | cmath
memory.h    | cmemory
stdio.h     | cstdio
stdlib.h    | cstdlib
string.h    | cstring
time.h      | ctime


## C++でC言語の関数を利用するには(2)

- C言語の関数名の前にstd::を付ける

```cpp
// NG:
printf("Hello World!\n");

// OK:
std::printf("Hello World!\n");
```




## 初期化リスト (1)

- 以下のメンバ初期化は誤り

```cpp
class Sample
{
private:
  int              value;
  std::vector<int> cache;
public:
  Sample(void) {
    value = 5;
    cache = std::vector<int>(25);
  }
  int getValue(void) {
    return this->value;
  }
  int getSize(void) {
    return this->cache.size();
  }
};
```


## 初期化リスト (2)

- 初期化リストを用いて初期化
  - メンバのコンストラクタの呼び出しが簡潔

```cpp
class Sample
{
private:
  int              value;
  std::vector<int> cache;
public:
  Sample(void) : value(5), cache(25) {
    // Do nothing
  }
  int getValue(void) {
    return this->value;
  }
  int getSize(void) {
    return this->cache.size();
  }
};
```




## メソッドの実装はクラス宣言外で行う (1)

- Javaみたいにクラス宣言内で定義をしない
- ただし，getter, setterであれば，クラス内で宣言してもよい
  - インライン展開される

```cpp
class Rect
{
private:
  double width;
  double height;
public:
  Rect(double _width, double _height) : width(_width), height(_height) {
    // Do nothing
  }
  // getter, setterは省略
  // これはクラス外で宣言するべき
  double calcArea(void) {
    if (width < 0 || height < 0) {
      throw "Bad width!";
    }
    return this->width * this->height;
  }
};
```


## メソッドの実装はクラス宣言外で行う (2)

- クラス宣言がスッキリした

```cpp
class Rect
{
private:
  double width;
  double height;
public:
  Rect(double _width, double _height) : width(_width), height(_height) {
    // Do nothing
  }
  double calcArea(void);
};

double Rect::calcArea(void) {
  if (width < 0 || height < 0) {
    throw "Bad width!";
  }
  return this->width * this->height;
}
```




## C++のキャスト

- C++においては，C言語のキャストを用いるべきでない
- 以下の4つのキャストを用いるべき
  1. `static_cast`
  2. `const_cast`
  3. `reinterpret_cast`
  4. `dynamic_cast`


## static_cast

- コンパイル時にエラーチェック
- int型からdouble型や，その逆

```cpp
int    foo = 10;
double bar = 3.14;

int    hoge = static_cast<int>(bar);
double hoge = static_cast<double>(foo);
```


## const_cast

- コンパイル時にエラーチェック
- CV修飾子を除去する
  - const
  - volatile
- 普通，用いることはない

```cpp
const char *STR = "Hello World!";
char *ptr = const_cast<int *>(STR);

// 代入可能
*ptr = 'A';
```


## reinterpret_cast

- コンパイル時にエラーチェック
- ポインタ間での型変換
  - これが主な用途
- 整数とポインタ(アドレス値)間の変換
  - この用途で用いることは，ほぼ無い

```cpp
int value = 0x00ffccff;
unsigned char *ptr = reinterpret_cast<unsigned char *>(&value);
```


## dynamic_cast

- 実行時にキャスト可能かどうかを判断
  - キャスト可能ならば，キャスト後のアドレスを返す
  - キャスト不可ならば，NULLを返す
- ポインタ型
- ダウンキャストを行いうときに用いる
  - 基本クラスから派生クラスへのキャスト




## C++でのメモリの動的確保 (1)

- `std::malloc()` ， `std::calloc()` ， `std::free()`は使わない
  - コンストラクタ，デストラクタが呼び出されない
- 代わりに``new` と `delete` を用いる
  - コンストラクタ，デストラクタが呼び出される


## C++でのメモリの動的確保 (2)

- クラス内で動的にメモリを確保する場合，コピーコンストラクタの実装は必須
  - デフォルトのコピーコンストラクタは，単にshallow copy を行うだけ
- これを怠ると，**SEGV**




## 定数マクロよりconst変数

- 定数マクロはナンセンス
- const変数を用いると，型検査が受けられる


## 関数マクロよりインライン関数

- 関数マクロの欠点
  - 型検査ができない
  - 副作用のある式を引数にとると，予想外の結果に
- インライン関数
  - 型検査可能
  - 副作用のある式を引数にとっても安心




## extern "C" (1)

- C++ではオーバーロード機能がある
- 関数名の一意性を保つため，名前マングルという処理をコンパイル時に行う
- C言語の遺産を再利用するときに困ることがある


## extern "C" (2)

簡単な例を示す

- util.c

```c
int mymax(int a, int b) {
  return a > b ? a : b;
}
```

- util.h

```c
#ifndef UTIL_H
#define UTIL_H

int mymax(int a, int b);

#endif
```


## extern "C" (3)

- main.cpp

```cpp
#include <iostream>
#include "util.h"

int main(void) {
  std::cout << mymax(2, 5) << std::endl;
  return 0;
}
```

- 以下のようにコンパイル

```sh
$ gcc -c util.c -o util.o
$ g++ -c main.cpp -o main.o
$ g++ main.o util.o -o main.out
```


## extern "C" (4)

- リンクエラーが発生する
  - `mymax(int, int)` が見つからないらしい

```txt
main.o:main.cpp:(.text+0x1f): undefined reference to `mymax(int, int)'
main.o:main.cpp:(.text+0x1f): relocation truncated to fit: R_X86_64_PC32
against undefined symbol `mymax(int, int)'
/usr/bin/ld: main.o: bad reloc address 0x0 in section `.pdata'
collect2: error: ld returned 1 exit status
```


## extern "C" (5)

- アセンブラを見てみると...
  - 以下，一部抜粋

```asm
main:

  ...

  call  __main
  movl  $5, %edx
  movl  $2, %ecx
  call  _Z5mymaxii
  movl  %eax, %edx
  movq  .refptr._ZSt4cout(%rip), %rcx
  call  _ZNSolsEi

  ...
```

- main()で参照される `mymax(int, int)` は `_Z5mymaxii` というシンボル名に
  なっている


## extern "C" (6)

単純な解決策

- util.c をC++コンパイラでコンパイルする
  - 本質的な解決策ではない

```sh
$ g++ -c util.c -o util.o
$ g++ -c main.cpp -o main.o
$ g++ main.o util.o -o main.out
```

- 問題点
  - ソースコードによっては，コンパイルに5分かかることもある
    - 再コンパイルするのは避けたい
  - C言語のオブジェクトファイルとして用いることができなくなる


## extern "C" (7)

本質的な解決策

- コンパイラにC言語の関数であることを伝える
  - そのための `extern "C"`
- ヘッダファイルに記述するだけでよい


## extern "C" (8)

書き方は以下の2通り

- 1つの関数に対して

```c
extern "C" mymax(int a, int b);
```

- 複数の関数に対して

```c
extern "C" {
mymax(int a, int b);
}
```


## extern "C" (9)

- `extern "C"` はC++の構文
  - C言語コンパイラでは認識されない
- 以下のように条件コンパイルを行う必要がある
  - マクロ `__cplusplus` が定義されているかどうかで判断
    - 定義されているならば，C++としてコンパイルしている
    - 定義されていないならば，C言語としてコンパイルしている

```c
#ifdef __cplusplus
extern "C" {
#endif

mymax(int a, int b);

#ifdef __cplusplus
}
#endif
```


## extern "C" (10)

- util.hを以下のように変更

```c
#ifndef UTIL_H
#define UTIL_H

#ifdef __cplusplus
extern "C" {
#endif

int mymax(int a, int b);

#ifdef __cplusplus
}
#endif

#endif
```

- 再コンパイル

```sh
$ g++ -c main.cpp -o main.o
$ g++ main.o util.o -o main.out
```


## extern "C" (11)

- main()のアセンブラを再度確認してみる

```asm
main:

  ...

  call  __main
  movl  $5, %edx
  movl  $2, %ecx
  call  mymax
  movl  %eax, %edx
  movq  .refptr._ZSt4cout(%rip), %rcx
  call  _ZNSolsEi

  ...
```

- 今度は，main()で参照される `mymax(int, int)` は `mymax` という
  シンボル名になっている
