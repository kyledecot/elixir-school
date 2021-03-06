---
layout: page
title: Erlangとの相互運用
category: advanced
order: 1
lang: jp
---

ErlangVMの上で開発することによって得られる利点の1つに、既にある大量のライブラリが利用できるという事があげられます。相互運用できることで、そうしたライブラリやErlangの標準ライブラリをElixirコードから活用することができます。このレッスンではサードパーティのErlangパッケージも併せ、標準ライブラリの関数へアクセスする方法を見ていきます。

{% include toc.html %}

## 標準ライブラリ

Erlangの豊富な標準ライブラリはアプリケーション内のどのElixirコードからもアクセスすることができます。Erlangモジュールは`:os`や`:timer`のように小文字のアトムで表されます。

`:timer.tc`を用いて、与えられた関数の実行時間を測定してみましょう:

```elixir
defmodule Example do
  def timed(fun, args) do
    {time, result} = :timer.tc(fun, args)
    IO.puts "Time: #{time}ms"
    IO.puts "Result: #{result}"
  end
end

iex> Example.timed(fn (n) -> (n * n) * n end, [100])
Time: 8ms
Result: 1000000
```

利用可能なモジュールの一覧は、[Erlang Reference Manual](http://erlang.org/doc/apps/stdlib/)を参照してください。

## Erlangパッケージ

以前のレッスンでMixと依存関係の管理を扱いましたが、Erlangライブラリを組み込むのも同様の方法で動作します。万が一Erlangライブラリが[Hex](https://hex.pm)に上がっていない(プッシュされていない)場合には、代わりにgitリポジトリを参照することができます:

```elixir
def deps do
  [{:png, github: "yuce/png"}]
end
```

これでErlangライブラリにアクセスできるようになりました:

```elixir
png = :png.create(#{:size => {30, 30},
                    :mode => {:indexed, 8},
                    :file => file,
                    :palette => palette}),
```

## 注目すべき違い

Erlangを用いる方法について理解したので、Erlangとの相互運用に伴って生じる直感的ではない部分についても扱うべきでしょう。

### アトム

ErlangのアトムはElixirのものにとてもよく似ていますが、コロン(`:`)がありません。小文字とアンダースコアで表されます:

Elixir:

```elixir
:example
```

Erlang:

```erlang
example.
```

### 文字列

Erlangの文字列はシングルクォート(`''`)で表され、Elixirの文字リストによく似ています。Erlangの文字列と記法を共用していることに加えて、文字リストのより一般的な用途はErlangコードと接続する際に用いられるというものです。

Elixir:

```elixir
"Example String"
```

Erlang:

```erlang
'Example String'.
```

重要なので注記しておくと、古いErlangライブラリではバイナリに対応していないものが多いため、Elixirの文字列は文字リストに変換する必要があります。ありがたいことに、これは`to_char_list/1`関数を用いて簡単に行うことができます:

```elixir
iex> :string.words("Hello World")
** (FunctionClauseError) no function clause matching in :string.strip_left/2
    (stdlib) string.erl:380: :string.strip_left("Hello World", 32)
    (stdlib) string.erl:378: :string.strip/3
    (stdlib) string.erl:316: :string.words/2

iex> "Hello World" |> to_char_list |> :string.words
2
```

### 変数

Elixir:

```elixir
iex> x = 10
10

iex> x1 = x + 10
20
```

Erlang:

```erlang
1> X = 10.
10

2> X1 = X + 1.
11
```

おしまいです！ ErlangをElixirアプリケーション内部から活用するのは簡単ですし、利用可能なライブラリの数が事実上倍になります。
