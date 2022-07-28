---
title: "実用Go言語読んだやつ その1"
description: ""
date: 2022-07-20T22:16:57+09:00
categories: "Tech"
tags: ["Go"]
draft: false
---

# 1章

## 変数名
- 頭字語の場合、全部大文字か、全部小文字にする。ほかの単語と組み合わせる場合は、全部大文字にする。
  - `ID, URL, id, url, ServeHTTP`
- エラーインタフェースを満たす型の名前には接尾辞`Error`をつける
  - `MarshalerError, UnsupportedTypeError`
- errors.New("")で宣言されるようなエラーの変数は`err`もしくは`Err`から始める
  - `ErrTooLong, ErrAdvanceTooFar`
- 基本的に変数名は短い名前が好まれる
  - for文のi、requestのreq
- しかし、グローバルだったりパッケージ外で呼ばれる変数は説明的であるほうがよい

## パッケージ名
- 小文字で構成される1つの単語が好ましい
  - `bytes, http, list`
- 汎用的すぎるパッケージ名は避けたい
  - `util, common, api`
- スネークケースやキャメルケースのパッケージ名はGoらしくないので、フォルダ分けるのがよい
  - `encoding_json -> encoding/json`
- パッケージは常に公開されているが、`internal`はモジュール外から読めない
- テストパッケージで`{name}_test`はあり

## 定数
- 配列、スライス、マップ、関数の返り値は定数として定義できない
```go
const (
    a = []int{1,2,3} // ダメ
    b = map[string]int{ // ダメ
        "one": 1,
        "tow": 2,
    }
    c = function() // ダメ
)
```

## エラー定数
- どうしても厳格にconstでErr定数を定義したい場合は以下のようにできる
```go
type errDatabase int
func (e errDatabase) Error() string {
    return "Database Error"
} 
const (
    ErrDatabase errDatabase = 0
)
```
- ここまで厳格にやる必要がなければ`var`で気軽に定義するのもあり
  - `databsae/sql`パッケージでは下記のように定義されている
  - しかし簡単に乗っ取り可能
```go
var ErrConnDone = errors.New("sql: connection is already closed")
ErrConnDone = errors.New("エラーを乗っ取ったぜ")
```

## 関数の引数
- たくさん引数がある場合、構造体を利用したオプション引数という手段がある
```go
type (
    Option struct {
        age int
        height int
        weight int
        health bool
    }

    Human struct {
        age int
        height int
        weight int
        health bool
    }
)

func NewHuman(opt Option) *Human {
    // 100キロ以上は不健康
    if opt.weight < 0 {
        opt.health = false
    }
    return &Human{
        age: opt.age,
        height: opt.height,
        weight: opt.weight,
        health: opt.health,
    }
}
```
- ビルダーの手法もある
  - 特定のフィールドの値を設定したオプション構造体を返すレシーバ関数を用意する
  - メソッドチェーンでつないでいく

## プログラムの引数
- コマンドライン引数は下記ライブラリを使われてる
  - 標準パッケージ`flag`
  - `gopkg.in/alecthomas/kingpin.v2`
  - `github.com/spf13/cobra`
- クラウドネイティブなアプリケーションにおいて、環境変数の利用が強く推奨されている
  - デプロイ時に環境変数を設定可能であるため、同じバイナリで柔軟に各環境向けの挙動を設定できる
  - `kelseyhightower/envconfig`

## メモリパフォーマンス
- スライスのメモリが足りなかった場合、2倍のサイズを確保してからそこに内容をコピーする
  - 1024までは2倍ずつメモリを確保する
- 長さがわかっている場合は`make()`でメモリを確保するべき
  - 確保とコピーの頻度を減らせる
- mapも同様に`make()`で確保するのがよい

## 文字列の結合
- 大量の文字列を結合する場合、`+`を使ってループで結合するとパフォーマンスが下がる
- Goにおいて文字列は不変なので、新しい文字列を生成するたびにメモリ確保が行われる
- `strings`パッケージの`strings.Builder`を利用するのがよい
  - 結合後のサイズがわかっているのであれば`Grow`でバッファサイズをしていするのがよい
```go
var builder strings.Builder
builder.Grow(100) // 100文字以下と仮定
for i, word := range src {
    if i != 0 {
        builder.WriteByte(' ') // byte配列として追加
    }
    builder.WriteString(word) // byte配列として追加
}
log.Println(builder.String()) // String()メソッドでstringにキャスト
```