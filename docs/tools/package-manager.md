# パッケージマネージャ
## 現存するC++のパッケージマネージャ

以下に示すのがC++のパッケージマネージャの中でも、ある程度規模の大きいものからそれ以上のものです。
それぞれインタフェースがかなり違うので、特に他の言語でパッケージマネージャを使用した経験がある人は違和感を感じるものもあると思います。

|                                             | 開発言語 | ビルドシステム | マルチプラットフォーム |
| ------------------------------------------- | ------- | ----------- | ------------------ |
| [poac](https://github.com/poacpm)           | C++     | :fa-check:  | :fa-check:         |
| [conan](https://conan.io)                   | Python  |             | :fa-check:         |
| [vcpkg](https://github.com/Microsoft/vcpkg) | CMake   |             | :fa-check:         |
| [buckaroo](https://buckaroo.pm)             | F#      | :fa-check:  | :fa-check:         |


## 使い方
それぞれのパッケージマネージャでREADME等に書かれている基本的な使い方を試してみます。

#### poac
> https://docs.poac.io/ja/getting-started/hello-world.html

```bash
$ poac new hello

Your "hello" project was created successfully.


Go into your project by running:
    $ cd hello

Start your project with:
    $ poac run

$ cd hello
$ poac run
Compiled: Output to `_build/bin/hello`
Running: `_build/bin/hello`
Hello, world!
```


#### conan
> Getting Startedが長すぎたので、ファイルの内容は省略した上で、なるべく短く単純な方法を作成しました。
本家の方法が気になる方は以下を参照して下さい。
https://docs.conan.io/en/latest/getting_started.html

```bash
$ mkdir hello
$ cd hello
$ nvim conanfile.txt
$ conan remote add conan-transit https://api.bintray.com/conan/conan/conan-transit
$ conan install . --build
$ nvim main.cpp
$ nvim CMakeLists.txt
$ cmake .
$ cmake --build .
$ ./bin/greet
Hello, world!
```


#### vcpkg
> conanと同じくファイルの内容は省略した上で、なるべく短く単純な方法を作成しました。
https://github.com/Microsoft/vcpkg/blob/master/docs/examples/installing-and-using-packages.md

```bash
$ ./vcpkg integrate install
$ ./vcpkg install poco
$ nvim main.cpp
$ nvim CMakeLists.txt
$ mkdir build
$ cd build
$ cmake ..
$ cmake --build .
$ ./main
```


#### buckaroo
> 注: 以下のコマンドを筆者が試したところ、かなり時間がかかったので試す際は注意してください。
また、以下の公式ドキュメントを参考にしましたが最後のコマンドが動作しませんでした。
https://github.com/LoopPerfect/buckaroo#readme

```bash
$ mkdir hello
$ cd hello
$ buckaroo init
$ buckaroo add github.com/buckaroo-pm/boost-thread@branch=master
$ buck run :my-app  # note: this command does not work
```


---

> C++のパッケージマネージャが他にもあるという場合や、その他間違いがあれば、[@matken11235](https://twitter.com/matken11235) までご連絡いただけると幸いです。
