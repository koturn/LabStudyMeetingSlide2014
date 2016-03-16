## CCILAB内勉強会 2回目

2016年3月16日




## 目次

- ネタ熱い内に実装せよ
- C++の話




## ネタ熱い内に実装せよ (1)

- 2016/3/10 21:00 のGIGAZINEの記事
    - 5秒以上タイピングをやめると全データが消滅するエディター
        - [The Most Dangerous Writing App](http://www.themostdangerouswritingapp.com/)
- 2016/03/10 22:23 に作られたリポジトリ
    - [vim-themostdangerouswritingapp](https://github.com/koturn/vim-themostdangerouswritingapp)


## ネタ熱い内に実装せよ (2)

- 実装しようと思った経緯
    1. キー操作しない時間が5秒続いたら自動削除ならVimでも実装できそう
    2. 「それVim！」
    3. とりあえず実装してウケを狙う


## ネタ熱い内に実装せよ (3)

- Vimにおけるタイマ実装
    - Vimにはタイマ機能は無い
    - キー操作しない時間が，`updatetime`に設定した時間経過後に以下のautocmdが発火することを利用
        - `CursorHold`
        - `CursorHoldI`
    - キー操作したときは，以下のイベントが発火する
        - `CursorMoved`
        - `CursorMovedI`
- `CursorHold`, `CursorHoldI` のタイミングでダミーのキー操作を発生させる

```vim
call feedkeys(mode() ==# 'i' ? "\<C-g>\<ESC>" : "g\<ESC>", 'n')
```


## ネタ熱い内に実装せよ (4)

- 操作しない時間を満了後に何かを行うようにするための概形は以下のような感じ
    - `updatetime` の値によっては，正確な時間計測はできない
    - 操作しない時間を満了後はイベント発火を起こさないようにする
        - `autocmd` の消去

```vim
function! timer#enable() abort
  call timer#disable()
  augroup timer
    autocmd CursorMoved,CursorMovedI * let s:timer_clock = 0
    autocmd CursorHold,CursorHoldI * call s:update()
  augroup END
  let s:timer_clock = 0
endfunction

function! timer#disable() abort
  unlet! s:timer_clock
  augroup timer
    autocmd! CursorHold,CursorHoldI,CursorMoved,CursorMovedI *
  augroup END
endfunction

function! s:update() abort
  if s:timer_clock < g:timer#time_to_stop
    call feedkeys(mode() ==# 'i' ? "\<C-g>\<ESC>" : "g\<ESC>", 'n')
    let s:timer_clock += &updatetime
  else
    " ここに時間満了後の処理を書く
    call timer#disable()
  endif
endfunction
```


## ネタ熱い内に実装せよ (5)

- `undo` 履歴は `:h clear-undo` に書かれている
    - Vimのヘルプのものは少し不十分なので，多少改良する
        - `normal` コマンドで再帰的展開を行わないようにする
        - ダミーのキー操作により，変更されたとしてVimに認識されないようにする
        - `undolevels` をローカルに変更するようにする

```vim
function s:clear_undo() abort
  let save_undolevels = &l:undolevels
  setlocal undolevels=-1
  execute "normal! a \<BS>\<Esc>"
  setlocal nomodified
  let &l:undolevels = save_undolevels
endfunction
```




![tsumugi_ccpp01.png](img/tsumugi_ccpp01.png)


![tsumugi_ccpp02.png](img/tsumugi_ccpp02.png)


## using namespace std の罠

- 前回サラッと紹介した事例
- 以下の2つのコードは実行結果が異なる
  - 何が異なるか？

```cpp
#include <cmath>
#include <iostream>
using namespace std;

int main() {
  cout << abs(2.1) << endl;  // abs(2.1) => 2.1
  return 0;
}
```

```cpp
#include <cmath>
#include <iostream>

int main() {
  std::cout << abs(2.1) << std::endl;  // abs(2.1) => 2
  return 0;
}
```


## using namespace std の罠

- C言語には2つのabs関数がそれぞれ異なる名前で提供されている
    - int型向けの `abs()`
    - double型向けの `fabs()`
- C++ では `abs()` は各型向けにオーバーロードされている
    - `abs()` の引数の型によって，int型向けにもdouble型向けにもなる
- 関数に渡された時点で暗黙的なキャストが行われている
    - `-Wall`, `-Wextra` でも検出不可
    - `-Wconversion` でようやく検出可能


## using namespace std の罠

```cpp
#include <cmath>
#include <iostream>
using namespace std;

int main() {
  cout << abs(2.1) << endl;  // abs(2.1) => 2.1
  return 0;
}
```

を `using namespace std` せずに書き直すと

```cpp
#include <cmath>
#include <iostream>

int main() {
  std::cout << std::abs(2.1) << std::endl;  // std::abs(2.1) => 2.1
  // std::cout << std::fabs(2.1) << std::endl;  // std::fabs(2.1) => 2.1
  return 0;
}
```


## using namespace std の罠

反対に

```cpp
#include <cmath>
#include <iostream>

int main() {
  std::cout << abs(2.1) << std::endl;  // abs(2.1) => 2
  return 0;
}
```

と同様のものを `using namespace std;` した場合に書き直すと

```cpp
#include <cmath>
#include <iostream>
using namespace std;

int main() {
  cout << ::abs(2.1) << endl;  // ::abs(2.1) => 2
  return 0;
}
```

どの名前空間に属さない `abs()` を呼び出すことを明示している




![meguru_static01.png](img/meguru_static01.png)


![meguru_static02.png](img/meguru_static02.png)


![meguru_static03.png](img/meguru_static03.png)


![meguru_static04.png](img/meguru_static04.png)


## C++ static

- C言語には2つのstaticがある
    - トップレベル（グローバル）でのstatic
    - 関数内でのstatic
- C++には加えて次の1つのstaticがある
    - クラス宣言内でのstatic


## トップレベルでのstatic (1)

- グローバル変数とか関数に付いてるやつ
- 以下のコード例のようなもの

```cpp
constexpr static double EPS = 1e-9;

static double
dbleq(double a, double b)
{
  return std::abs(a - b) < EPS;
}
```


## トップレベルでのstatic (2)

- この翻訳単位でしか使わないという表明
    - 翻訳単位とは，ざっくり言えば1ソースファイル
    - 正確には，プリプロセスによって，ヘッダファイルの中身をぶちまけた後の1ソースファイル
    - コンパイルによって，適当なシンボル名に置き換わり，他の翻訳単位からの直接的な呼び出しは不可能になってくれる
        - 他の翻訳単位の同一の名前空間でも，同じ名前の関数，変数を利用できる
- コンパイラが最適化を行うヒントになる
    - この翻訳単位内でしか使わないなら，関数の実体を削除してもいいよねという判断
        - gcc/g++ の `-finline-functions` （`-O3` に含まれる）
    - `inline` 指定と併用される
        - ヘッダファイルに書いた `inline` 関数にはstatic指定必須
- 可読性の面でも，ソースコード読者に優しい
    - この翻訳単位でしか使わないことがわかるから


## トップレベルでのstatic (おまけ話 1)

- ちょっと横道に沿れた話
    - `inline` 指定された関数をインライン展開するかどうかはコンパイラの判断次第
    - コンパイラ拡張によって，より強制度の高い `inline` 指定を利用することも可能
        - 通常，ここまで強いインライン展開を指定する必要はない

指定の仕方                              | 効果
----------------------------------------|----------------------------------------------------------
`__forceinline`                         | かなり強制力の高い `inline` 指定 (MSVC, Intel C Compiler)
`__attribute__((always_inline)) inline` | 常にインライン展開を行う (gcc, g++, clang)


## トップレベルでのstatic (おまけ話 2)

- こんな感じにマクロ化する

```cpp
#if defined(__GNUC__)
#  define FORCEINLINE __attribute__((always_inline)) inline
#elif defined(_MSC_VER) || defined(__INTEL_COMPILER)
#  define FORCEINLINE  __forceinline
#elif defined(__STDC_VERSION__) && __STDC_VERSION__ >= 199901L || defined(__cplusplus)
#  define FORCEINLINE  inline
#else
#  define FORCEINLINE
#endif

static FORCEINLINE
extgcd(int a, int b, int& x, int& y)
{
  int v = x = 0;
  int u = y = 1;
  while (a != 0) {
    int q = b / a;
    std::swap(x -= q * u, u);
    std::swap(y -= q * v, v);
    std::swap(b -= q * a, a);
  }
  return b;
}
```


## 関数内でのstatic (1)

- ややクセモノといえる
- 使用用途としては2つ考えられる

```cpp
static int
cntFunction()
{
  static int cnt = 0;
  return cnt++;
}
```

```cpp
int
main()
{
  static int array[5000000];
  return 0;
}
```


## 関数内でのstatic (2)

- 使用用途
    - 関数内の変数の寿命を永続化させる
    - 巨大な配列等をスタック領域から静的領域に移すため
        - 静的領域なら，多少大きめの容量でも確保できる
    - 同じく静的領域に配置されるグローバル変数にしないのは，スコープを限定するため

```cpp
// このコードは多分実行した瞬間に死ぬ
// static付けると大丈夫なはず
int
main()
{
  int array[5000000];
  return 0
}
```


## 関数内でのstatic (3)

- 変数に使われる領域の大別
    - 静的領域
        - グローバル変数と関数内static変数
        - プログラムの開始時から終了時まで寿命がある
        - 明示的な初期化をしなくても，整数配列等は0で初期化されている
    - スタック領域
        - ローカル変数に使われる領域
        - 関数の1コール毎に成長するスタックに含まれる
        - デカい容量を取ることはできない
            - 特に組み込み系だと取れる領域は小さいので，配列にstaticはほぼ必須
        - 明示的に初期化しないとゴミが入っている
    - ヒープ領域
        - `new` や `std::malloc()` などの動的確保によって使われる領域


## クラス宣言内でのstatic

- インスタンスではなく，クラスにメンバ変数，メンバ関数が属することの表明
    - `className::memberFunction()` という形で呼び出す
    - Javaと同じ

```cpp
// declare
class Hoge
{
public:
  static int a;
  static int func();
};  // class Hoge

// implementation
int Hoge::a = 100;

int
Hoge::func()
{
  return 42;
}
```




![nene_cout01.png](img/nene_cout01.png)


![nene_cout02.png](img/nene_cout02.png)


![nene_cout03.png](img/nene_cout03.png)


![nene_cout04.png](img/nene_cout04.png)


## 自作クラスをcout / cinに対応させる (1)

- `std::cout` は様々な型に対応しており，適当に渡せばよしなにやってくれる
    - `std::printf()` はフォーマット文字列が必要だった
- これを自作クラスに対応させたい
    - モチベーションとしては，Javaにおける `toString()` メソッドのオーバーライドのようなことをしたい


## 自作クラスをcout / cinに対応させる (2)

- 基本的には以下のようにすれば `cout` に対応できる
    - `friend` 指定はprivateメンバにアクセスできるようにするため
    - ノリとしては，Javascriptによくあるメソッドチェーン
        - 自身を返却する

```cpp
#include <iostream>
#include <string>

class Hoge
{
private:
  int a;
  std::string s;
public:
  Hoge();
  friend std::ostream& operator<<(std::ostream& os, const Hoge& this_);
};  // class Hoge

Hoge::Hoge() :
  a(42),
  s("Hello")
{}

std::ostream&
operator<<(std::ostream& os, const Hoge& this_)
{
  os << this_.a << " : " << this_.s;
  return os;
}

int
main()
{
  Hoge h;
  std::cout << h << std::endl;  // => 42 : Hello
  return 0;
}
```


## 自作クラスをcout / cinに対応させる (3)

- `ostream` 周りをテンプレートで書く例もある

```cpp
#include <iostream>
#include <string>

class Hoge
{
private:
  int a;
  std::string s;
public:
  Hoge();

  template<typename CharT, typename Traits>
  friend std::basic_ostream<CharT, Traits>&
  operator<<(std::basic_ostream<CharT, Traits>& os, const Hoge& this_);
};  // class Hoge

Hoge::Hoge() :
  a(42),
  s("Hello")
{}

template<typename CharT, typename Traits>
std::basic_ostream<CharT, Traits>&
operator<<(std::basic_ostream<CharT, Traits>& os, const Hoge& this_)
{
  os << this_.a << " : " << this_.s;
  return os;
}

int
main()
{
  Hoge h;
  std::cout << h << std::endl;  // => 42 : Hello
  return 0;
}
```


## 自作クラスをcout / cinに対応させる (4)

- プライベートメンバへのアクセスが不要なら，`friend` 指定は必要ない

```cpp
#include <iostream>
#include <string>

class Hoge
{
public:
  int a;
  std::string s;

  Hoge();
};  // class Hoge

Hoge::Hoge() :
  a(42),
  s("Hello")
{}

std::ostream&
operator<<(std::ostream& os, const Hoge& this_)
{
  return os << this_.a << " : " << this_.s;
}

int
main()
{
  Hoge h;
  std::cout << h << std::endl;
  return 0;
}
```


## 自作クラスをcout / cinに対応させる (5)

- cinも同じノリで演算子オーバーロードできる

```cpp
#include <iostream>
#include <string>

class Hoge
{
private:
  int a;
  std::string s;
public:
  Hoge();

  friend std::ostream&
  operator<<(std::ostream& os, const Hoge& this_);

  friend std::istream&
  operator>>(std::istream& is, Hoge& this_);
};  // class Hoge

Hoge::Hoge() {}  // do nothing

std::ostream&
operator<<(std::ostream& os, const Hoge& this_)
{
  return os << this_.a << " : " << this_.s;
}

std::istream&
operator>>(std::istream& is, Hoge& this_)
{
  is >> this_.a >> this_.s;
  return is;
}

int
main()
{
  Hoge h;
  std::cin >> h;  // $ echo 114514 World | ./a.out と実行
  std::cout << h << std::endl;  // => 114514 : World
  return 0;
}
```


## std::vector をcoutで表示

- `std::vector` の全要素をcoutで表示する

```cpp
#include <iostream>
#include <vector>

template<typename T>
std::ostream&
operator<<(std::ostream& os, const std::vector<T>& vct)
{
  for (const auto& e : vct) {
    os << e << ", ";
  }
  return os;
}

int
main()
{
  std::vector<int> vct{1, 1, 4, 5, 1, 4};
  std::cout << vct << std::endl;  // => 1, 1, 4, 5, 1, 4,
  return 0;
}
```




![touko_option01.png](img/touko_option01.png)


![touko_option02.png](img/touko_option02.png)


## `-g3`

- `-g` はgdbを使うためのオプションとして有名
- マクロの定義をデバッグ用実行ファイルに残すには `-g3` を指定するとよい
- 最適化しないようにオプション `-O0` も指定しておく


## `-D_FORTIFY_SOURCE=2`

- `-D_FORTIFY_SOURCE=2`
    - 文字列やメモリーの操作を行う様々なC言語関数を使用する際に，バッファオーバーフローを検出するチェックを行う
    - コンパイラオプションでマクロ `_FORTIFY_SOURCE` を定義しているだけ
        - `#define _FORTIFY_SOURCE 2` と同じ


## `-D_GLIBCXX_DEBUG`

- `-D_GLIBCXX_DEBUG`
    - STLコンテナの範囲外アクセスの検出ができるようにする
    - コンパイラオプションでマクロ `_GLIBCXX_DEBUG` を定義しているだけ
        - STLコンテナ関連のヘッダをインクルードする前に `#define _GLIBCXX_DEBUG` するのは微妙

```cpp
#include <iostream>
#include <vector>

int
main()
{
  std::vector<int> vct{1, 1, 4, 5, 1, 4}
  std::cout << vct[1] << std::endl;
  vct[6] = 1;
  std::cout << vct[6] << std::endl;
  return 0;
}
```

```txt
/usr/local/gcc-head/include/c++/6.0.0/debug/vector:415:
Error: attempt to subscript container with out-of-bounds index 6, but
container only holds 6 elements.

Objects involved in the operation:
    sequence "this" @ 0x0x7ffd676df690 {
      type = std::__debug::vector<int, std::allocator<int> >;
    }
```


## `-ftrapv`

- `-ftrapv`
    - 符号有り整数同士での加算，減算，乗算のオーバーフローを検出する
        - 検出した場合，その時点でabortする
        - 符号無し整数のオーバーフローの検出は不可

```cpp
#include <iostream>
#include <limits>

int
main()
{
  std::cout << (std::numeric_limits<int>::max() + 1) << std::endl;
  std::cout << (std::numeric_limits<int>::min() - 1) << std::endl;
  return 0;
}
```


## `-fstack-protector-all` (1)

- `-fstack-protector-all`
    - スタック破壊検出コードを生成
        - 検出時にバックトレースを表示し，abort
    - libssp.aをリンクする必要がある（`-lssp` と併用する必要がある）
    - 範囲外アクセスを検出できるわけではない
        - バッファオーバーフローによるリターンアドレスの書き換えなどを検出するためのもの
        - 関数に関与しない領域外の書き換えは検出不可

```cpp
#include <iostream>

int
main()
{
  int a[10] = {};
  std::cout << a[5] << std::endl;
  a[10] = 12;  // a[300] は関数の管理領域外なので，おそらく検出不能
  std::cout << a[10] << std::endl;
  return 0;
}
```


## `-fstack-protector-all` (2)

```txt
$ g++ -std=gnu++11 -fstack-protector-all a.cpp
$ ./a.out
0
12
*** stack smashing detected ***: ./prog.exe terminated
======= Backtrace: =========
/lib/x86_64-linux-gnu/libc.so.6(__fortify_fail+0x37)[0x7fb165799e57]
/lib/x86_64-linux-gnu/libc.so.6(__fortify_fail+0x0)[0x7fb165799e20]
./prog.exe[0x400d4f]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xed)[0x7fb1656b176d]
./prog.exe[0x400bc9]
======= Memory map: ========
00400000-00401000 r-xp 00000000 fd:01 4992842                            /home/jail/prog.exe
00601000-00602000 rw-p 00001000 fd:01 4992842                            /home/jail/prog.exe
00f39000-00f6b000 rw-p 00000000 00:00 0                                  [heap]
7fb162e56000-7fb162e58000 r-xp 00000000 fd:01 11933814                   /lib/x86_64-linux-gnu/libdl-2.15.so
7fb162e58000-7fb163058000 ---p 00002000 fd:01 11933814                   /lib/x86_64-linux-gnu/libdl-2.15.so
7fb163058000-7fb163059000 r--p 00002000 fd:01 11933814                   /lib/x86_64-linux-gnu/libdl-2.15.so
7fb163059000-7fb16305a000 rw-p 00003000 fd:01 11933814                   /lib/x86_64-linux-gnu/libdl-2.15.so
7fb16305a000-7fb1646c5000 r--p 00000000 fd:01 8126494                    /usr/local/lib/libicudata.so.52.1
7fb1646c5000-7fb1648c4000 ---p 0166b000 fd:01 8126494                    /usr/local/lib/libicudata.so.52.1
7fb1648c4000-7fb1648c5000 rw-p 0166a000 fd:01 8126494                    /usr/local/lib/libicudata.so.52.1
7fb1648c5000-7fb1648db000 r-xp 00000000 fd:01 11927780                   /lib/x86_64-linux-gnu/libz.so.1.2.3.4
7fb1648db000-7fb164ada000 ---p 00016000 fd:01 11927780                   /lib/x86_64-linux-gnu/libz.so.1.2.3.4
7fb164ada000-7fb164adb000 r--p 00015000 fd:01 11927780                   /lib/x86_64-linux-gnu/libz.so.1.2.3.4
7fb164adb000-7fb164adc000 rw-p 00016000 fd:01 11927780                   /lib/x86_64-linux-gnu/libz.so.1.2.3.4
...
```


## `-fsanitize=address` (1)

- 範囲外アクセスの検出
- 動的確保した配列に対しても有効
- `-fno-omit-frame-pointer` と併用する必要がある
- Cygwinとかでは利用不可

```cpp
#include <iostream>
#include <memory>

int
main()
{
  std::unique_ptr<int[]> ptr(new int[10]);
  ptr[5] = 10;
  std::cout << ptr[5] << std::endl;
  ptr[13] = 10;
  std::cout << ptr[13] << std::endl;
  return 0;
}
```


## `-fsanitize=address` (2)

- 実行結果

```txt
$ g++ -std=gnu++11 -fsanitize=address -fno-omit-frame-pointer a.cpp
$ ./a.out
10
=================================================================
==721== ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60080000c004 at pc 0x400e48 bp 0x7ffe4eeddb20 sp 0x7ffe4eeddb18
WRITE of size 4 at 0x60080000c004 thread T0
    #0 0x400e47 (/home/koturn/workspace/a.out+0x400e47)
    #1 0x7f6344ac2ec4 (/lib/x86_64-linux-gnu/libc-2.19.so+0x21ec4)
    #2 0x400c28 (/home/koturn/workspace/a.out+0x400c28)
0x60080000c004 is located 12 bytes to the right of 40-byte region [0x60080000bfd0,0x60080000bff8)
allocated by thread T0 here:
    #0 0x7f634539188a (/usr/lib/x86_64-linux-gnu/libasan.so.0.0.0+0x1188a)
    #1 0x400d33 (/home/koturn/workspace/a.out+0x400d33)
    #2 0x7f6344ac2ec4 (/lib/x86_64-linux-gnu/libc-2.19.so+0x21ec4)
Shadow bytes around the buggy address:
  0x0c017fff97b0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff97c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff97d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff97e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff97f0: fa fa fa fa fa fa fa fa fa fa 00 00 00 00 00 fa
=>0x0c017fff9800:[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff9810: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff9820: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff9830: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff9840: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c017fff9850: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:     fa
  Heap righ redzone:     fb
  Freed Heap region:     fd
  Stack left redzone:    f1
  Stack mid redzone:     f2
  Stack right redzone:   f3
  Stack partial redzone: f4
  Stack after return:    f5
  Stack use after scope: f8
  Global redzone:        f9
  Global init order:     f6
  Poisoned by user:      f7
  ASan internal:         fe
==721== ABORTING
```


## 紹介したデバッグオプションのまとめ

デバッグオプション                                   | 意味
-----------------------------------------------------|----------------------------------------
`-g3` （と `-O0`）                                   | gdbでのデバッグ用
`-D_FORTIFY_SOURCE=2`                                | C言語関数のバッファオーバーフローの検出
`-D_GLIBCXX_DEBUG`                                   | STLの範囲外アクセスなどの検出を有効に
`-ftrapv`                                            | 整数同士の演算のオーバーフローの検出
`-fstack-protector-all` と `-fno-omit-frame-pointer` | スタック破壊を検出
`-fsanitize=address`                                 | 領域外アクセスの検出




![meguru_exception01.png](img/meguru_exception01.png)


![meguru_exception02.png](img/meguru_exception02.png)


![meguru_exception03.png](img/meguru_exception03.png)


## 例外を投げる (1)

- 適当なサイトを参考にすると，文字列リテラル（const char*）を投げていることがある

```cpp
#include <iostream>

int
tes(int a)
{
  if (a <= 0) {
    throw "An error occured";
  }
  return a - 1;
}

int
main()
{
  try {
    std::cout << tes(10) << std::endl;
    std::cout << tes(0) << std::endl;
  } catch (const char* errmsg) {
    std::cerr << errmsg << std::endl;
  }
  return  0;
}
```


## 例外を投げる (2)

- 文字列を投げるのはナンセンス
- せめて，`<stdexcept>` の例外にラップして，エラーメッセージを投げる
    - メッセージは例外オブジェクトの `what()` メンバ関数で取得

```cpp
#include <iostream>
#include <stdexcept>

int
tes(int a)
{
  if (a <= 0) {
    throw std::runtime_error("An error occured");
  }
  return a - 1;
}

int
main()
{
  try {
    std::cout << tes(10) << std::endl;
    std::cout << tes(0) << std::endl;
  } catch (const std::exception& e) {
    // std::exception は std::runtime_error の親クラス
    std::cerr << e.what() << std::endl;
  }
  return  0;
}
```


## 例外を投げる (3)

- Javaみたいに例外を投げた箇所を知りたい
    - 途中の関数のスタックトレースを取ることは難しいが，投げた箇所を限定するなら可能

```cpp
#include <iostream>
#include <stdexcept>

#if defined(_MSC_VER)
#  define __DBG_FUNCTION__  __FUNCSIG__
#elif defined(__GNUC__)
#  define __DBG_FUNCTION__  __PRETTY_FUNCTION__
#else
#  define __DBG_FUNCTION__  __func__
#endif

#define MACRO_EXPAND(x)  MACRO_STR(x)
#define MACRO_STR(x)  #x

int
tes(int a)
{
  if (a <= 0) {
    throw std::runtime_error(
        "File: " __FILE__ "
        at line " MACRO_EXPAND(__LINE__) " " +
        std::string(__DBG_FUNCTION__) + "> An error occured");
  }
  return a - 1;
}
```


## 例外を投げる (4)

- `main()` 関数はこんな感じで

```cpp
int
main()
{
  try {
    std::cout << tes(10) << std::endl;
    std::cout << tes(0) << std::endl;
  } catch (const std::exception& e) {
    // std::exception は std::runtime_error の親クラス
    std::cerr << e.what() << std::endl;
  }
  return  0;
}
```

- 実行結果 (g++)

```cpp
File: sample.cpp at line 19 int tes(int)> An error occured
```




![touko_ansiescape01.png](img/touko_ansiescape01.png)


![touko_ansiescape02.png](img/touko_ansiescape02.png)


## 例外を投げる (5)

- 文字色を付けたいなら，以下のように投げる
    - ANSIエスケープシーケンス対応のターミナルエミュレータでないと不可
    - リダイレクトすると，特殊文字はそのまま残ってしまう

```cpp
throw std::runtime_error(
    "\x1b[31mFile: " __FILE__ "
    at line " + std::to_string(__LINE__) + " " +
    std::string(__DBG_FUNCTION__) + "> An error occured\x1b\039[31m");
```


![touko_ansiescape03.png](img/touko_ansiescape03.png)


## ANSI エスケープシーケンス（前景色）

- 文字（前景色）に色を付けるエスケープシーケンスは以下の通り
    - EscはC/C++では `'\033'` もしくは `'\x1b'` とする

エスケープシーケンス | 色
---------------------|-----------------
`Esc[30m`            | 黒
`Esc[31m`            | 赤
`Esc[32m`            | 緑
`Esc[33m`            | 黄色
`Esc[34m`            | 青
`Esc[35m`            | マゼンタ
`Esc[36m`            | シアン
`Esc[37m`            | 白
`Esc[39m`            | デフォルトに戻す


## ANSI エスケープシーケンス（背景色）

エスケープシーケンス | 色
---------------------|-----------------
`Esc[40m`            | 黒
`Esc[41m`            | 赤
`Esc[42m`            | 緑
`Esc[43m`            | 黄色
`Esc[44m`            | 青
`Esc[45m`            | マゼンタ
`Esc[46m`            | シアン
`Esc[47m`            | 白
`Esc[49m`            | デフォルトに戻す





![meguru_macro01.png](img/meguru_macro01.png)


![meguru_macro02.png](img/meguru_macro02.png)


## 特殊なマクロと定数 (1)

- 以下の2つは特殊なマクロ

特殊なマクロ | 展開結果
-------------|---------------------------------------------------------
`__FILE__`   | ファイル名に展開される（文字列リテラルになる）
`__LINE__`   | 現在の行数に展開される（ダブルクオートで囲まれていない）


## 特殊なマクロと定数 (2)

- 以下は特殊なマクロや定数
    - `__FUNCTION__` はMSVCならクラス名も含み，g++は含まない

定数名                |
----------------------|-------------------------------------------------------------------------------
`__PRETTY_FUNCTION__` | シグネチャを含めた現在位置の関数の情報を得る（**gcc**の定義済み**定数**）
`__FUNCSIG__`         | シグネチャを含めた現在位置の関数の装飾名を得る（**MSVC**の定義済み**マクロ**）
`__FUNCDNAME__`       | シグネチャを含めた現在位置の関数の情報を得る（**MSVC**の定義済み**マクロ**）
`__func__`            | 現在位置の関数名に展開される（クラス名は含まない）．C99で実装された**定数**
`__FUNCTION__`        | gccとMSVCのどちらにもあるコンパイラ拡張．現在位置の関数名を得る（**定数**）



## 特殊なマクロと定数 (3)

- 以下のコードをg++でコンパイルし，実行

```cpp
#include <iostream>

class Test {
public:
  void test02() {
    std::cout << "__func__: " << __func__ << std::endl;
    std::cout << "__FUNCTION__: " << __FUNCTION__ << std::endl;
    std::cout << "__PRETTY_FUNCTION__: " << __PRETTY_FUNCTION__ << std::endl;
  }
};  // class Test

void test01() {
  std::cout << "__func__: " << __func__ << std::endl;
  std::cout << "__FUNCTION__: " << __FUNCTION__ << std::endl;
  std::cout << "__PRETTY_FUNCTION__: " << __PRETTY_FUNCTION__ << std::endl;
}

int
main()
{
  test01();
  std::cout << "========================================" << std::endl;
  Test t;
  t.test02();
  return 0;
}

```


## 特殊なマクロと定数 (4)

- 実行結果

```txt
__func__: test01
__FUNCTION__: test01
__PRETTY_FUNCTION__: void test01()
========================================
__func__: test02
__FUNCTION__: test02
__PRETTY_FUNCTION__: void Test::test02()
```


## 特殊なマクロと定数 (5)

- 以下のコードをMSVCでコンパイルし，実行

```cpp
#include <iostream>

class Test {
public:
  void test02() {
    std::cout << "__func__: " << __func__ << std::endl;
    std::cout << "__FUNCTION__: " << __FUNCTION__ << std::endl;
    std::cout << "__FUNCSIG__: " << __FUNCSIG__ << std::endl;
    std::cout << "__FUNCDNAME__: " << __FUNCDNAME__ << std::endl;
  }
};  // class Test

void test01() {
  std::cout << "__func__: " << __func__ << std::endl;
  std::cout << "__FUNCTION__: " << __FUNCTION__ << std::endl;
  std::cout << "__FUNCSIG__: " << __FUNCSIG__ << std::endl;
  std::cout << "__FUNCDNAME__: " << __FUNCDNAME__ << std::endl;
}

int
main()
{
  test01();
  std::cout << "========================================" << std::endl;
  Test t;
  t.test02();
  return 0;
}
```


## 特殊なマクロと定数 (6)

- MSVCでの実行結果

```txt
__func__: test01
__FUNCTION__: test01
__FUNCSIG__: void __cdecl test01(void)
__FUNCDNAME__: ?test01@@YAXXZ
========================================
__func__: test02
__FUNCTION__: Test::test02
__FUNCSIG__: void __cdecl Test::test02(void)
__FUNCDNAME__: ?test02@Test@@QEAAXXZ
```


## 特殊なマクロと定数 (6)

- `__LINE__` は文字列リテラルではない
    - `42` のように，ダブルクオート無しの形に展開される
- 以下のようにすることで，プリプロセス時にダブルクオート付加
    - ポイントはマクロにある文字列化演算子 `#`
- （`std::to_string(__LINE__)`  として，`+` で結合すればいいのはナイショ）

```cpp
#define MACRO_EXPAND(x)  MACRO_STRING(x)
#define MACRO_STRING(x)  #x
```


## 特殊なマクロと定数 (7)

- C言語の仕様では，空白区切りの文字列リテラルはコンパイル時に結合される

```cpp
#include <iostream>

int
main()
{
  std::cout << "Hello World!" << std::endl;
  std::cout << "Hello" " World!" << std::endl;
  return 0;
}
```




![meguru_debug01.png](img/meguru_debug01.png)


![meguru_debug02.png](img/meguru_debug02.png)


## プリントデバッグ用マクロ (1)

- こんな感じで定義するとよさそう

```cpp
#include <iostream>
#include <string>

#if defined(_MSC_VER)
#  define __DBG_FUNCTION__  __FUNCSIG__
#elif defined(__GNUC__)
#  define __DBG_FUNCTION__  __PRETTY_FUNCTION__
#else
#  define __DBG_FUNCTION__  __func__
#endif

#define DBGLOG(...) dbglog(__FILE__, __LINE__, __DBG_FUNCTION__, ##__VA_ARGS__)

void
dbglog(const char* filename, unsigned int linenr, const char* funcsig, const std::string& msg="")
{
  std::cerr << "File: " << filename
            << " at line " << linenr
            << ", " << funcsig;
  if (msg != "") {
    std::cerr << "> " << msg;
  }
  std::cerr << std::endl;
}
```


## プリントデバッグ用マクロ (2)

- main側はこんな感じで

```cpp
int
main()
{
  DBGLOG();
  DBGLOG("Hello");
  return 0;
}
```

- 実行結果

```txt
File: prog.cc at line 46, int main()
File: prog.cc at line 48, int main()> Hello
```


## プリントデバッグ用マクロ (3)

- 変数の中身を `x = 10` みたいな形で表示したいなら，以下のようにする

```cpp
// ...略

#define DBGDUMP(var) dbgdump(__FILE__, __LINE__, __DBG_FUNCTION__, #var, var)

template<typename T>
void
dbgdump(const char* filename, unsigned int linenr, const char* funcsig, const char* varname, T var)
{
  std::cerr << "File: " << filename
            << " at line " << linenr
            << ", " << funcsig
            << "> " << varname
            << " = " << var;
            << std::endl;
}

int
main()
{
  int a = 42;
  // File: prog.cc at line 46, int main()> a = 42 のように表示される
  DBGDUMP(a);
  return 0;
}
```
