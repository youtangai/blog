---
title: "Akka Streamsについて調べてみたよ"
description: "チームでブラックボックスになっているシステムで利用されているAkka Streamsについて調べました。"
featured_image: "/images/akka_stream.png"
images: ["/images/akka_stream.png"]
date: 2021-12-07T21:19:00+09:00
categories: "Tech"
tags: ["Akka Stream"]
---

# はじめに
皆さんこんにちは、最近パーソナルトレーニングを始めたyoutangaiです。
ダイエットと運動不足解消を目的に始めたのですが、適切なフォームでトレーニングできてとても満足度が高いです。

さて、今回の記事は、メディア事業部の広告横軸組織PTA([twitter](https://twitter.com/PTA_CyberAgent))の[アドベントカレンダー](https://adventar.org/calendars/6450)8日目の記事となります。よろしくお願い致します。

先に述べますと今回の内容はAkka Streamsの基礎的な概念を調査した内容になります。そのため、対象の読者は私と同じで`Akka Streamsなんもわからん`という方々です。

# 動機
Amebaの広告システムは稼働してから7,8年くらい(正確じゃないかもです)経過しており、古くから稼働しているシステムが結構あります。
その中でも、RDBに保存されているレコードをインメモリデータストアに保存するシステムが、ブラックボックスになってしまっています。
我々の広告配信システムにおいてかなり重要なシステムで、ここで障害が発生すると大損害が発生する可能性があります。
このままではまずいということで、このシステムについて調査することになりました。
しかし、このシステムはScalaのAkka Streamsで実装されており、ソースコードを読んでいてもすんなり理解できる内容ではありませんでした。
そこで、基礎的な部分からしっかり理解しようと思ったのが動機です。

# Akka Streamsとは
ここからが、本題です。\
Akka Streamsとは、[Reactive Streams](https://www.reactive-streams.org/)のAkka実装です。
そのため、Akka Streamsを理解するためにはReactive Streamsを理解する必要があります。

## Reactive Streams
Reactive Streamsのドキュメントではこのように記述されています。
```
Reactive Streams is an initiative to provide a standard for asynchronous stream processing with non-blocking back pressure
```
直訳すると、Reactive Streamsとは`ノンブロッキング`な`バックプレッシャー`を備えた`非同期ストリーム処理の標準`です。
ノンブロッキングや非同期ストリーム処理はなんとなく想像がつくのですが、バックプレッシャーという単語は初耳でした。

## バックプレッシャー
Reactive Streamsでは、データを流す`パブリッシャー`と、流れてきたデータを受け取る`サブスクライバー`が登場します。

バックプレッシャーがない場合、パブリッシャーが流すデータの量がサブスクライバーの処理能力を上回ると、サブスクライバーの処理が間に合わず、データが詰まってしまいます。

このような問題を回避するためにバックプレッシャーがあります。サブスクライバーは、自身が処理できるデータ量を定義し、自身が処理できる分だけパブリッシャーからデータを受け取ります。パブリッシャー側で流量制限を行う必要がないため、効率よくデータを処理することが可能になります。

# 抽象概念について
さて、これまでReactive Streamsについて述べてきましたが、ここからはAkka Streamsにおける概念について述べます。
Akka Streamsには`Source`、`Sink`、`Flow`、`Graph`という4つの重要な抽象化された概念があります。先に簡単に述べてしまうと、Sourceでデータを作成し、Flowで流れてきたデータを加工し、Sinkで最終的に加工されたデータを受け取るという流れになっています。これらの流れ全体をGraphと呼びます。

## Source
データを生成する場所、つまりパブリッシャーです。ここで、ファイルの中身を読み出したり、HTTPリクエストボディの中身を読み出したり、データストアからデータを読み出したりする。

## Sink
最終的にデータを受け取る場所、つまりサブスクライバーです。ここで、受け取ったデータをファイルに書き出したり、データストアに書き込んだりするはず。実際にファイル書き出ししている例は見つけたが、データストアに書き込んだりする例を見つけられなかったので想像の話をしています。例があったら教えて下さい。

## Flow
データを処理する場所です。Flowだけは特殊で、サブスクライバーでありパブリッシャーです。Sourceからデータを受け取ったり、FlowからFlowへデータが流れたり、FlowからSinkにデータを流したりします。
ここでは、流れてきたデータのデコード処理だったり、想定しないデータのフィルタリングなどを行います。

## Graph
Sourceに始まり、0以上のFlowを経由して、Sinkに終わる一連の流れがGraphです。
このGraphを定義し実行することで、処理されたデータを得ることができます。

# おわりに
今回調べた結果、ソースコードを読んでいてわからないと思っていた部分が理解できたので、今後のシステム調査はスムーズに進んでくれるといいなと思っています。
引き続きシステムの調査は行うので、改めて調査が必要な内容が出てきたら再度まとめたいと考えています。

最後まで読んでいただきありがとうございました。

# 参考文献
- "Reactive Streams". https://www.reactive-streams.org/
- Xiao Yang. "Akka Streamsについての基礎概念". Qiita. 2018/02/01. https://qiita.com/xoyo24/items/299ee3e624f4afe2d27a, (Accessed: 2021/12/07)
- 前出祐吾. "Akka Streamsで実装するリアクティブストリーム"._ Think IT. 2018/03/07. https://thinkit.co.jp/article/13485, (Accessed: 2021/12/07)