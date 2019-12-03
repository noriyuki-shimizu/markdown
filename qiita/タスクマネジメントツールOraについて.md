# タスクマネジメントツール Ora がよかったって話

hash: Ora task Management

## はじめに

皆さんのタスクマネジメントツールは何を使用していますでしょうか？
僕は、そこらへんあんま詳しくないので Redmine とか Jira とか Trello とか ZenHub くらいしかわかりません。
それぞれ特徴とかあるとは思います。システム開発がウォーターフォールモデルなら Redmine 、アジャイル開発なら Jira など
自分が今回出会ったタスクマネジメントツールは Ora と言う、アジャイル開発向けのなかなかイケてるものです。
実際に使ってみて、もちろん使いにくいところは多少あったりもしますが、多彩な機能があります。個人的な意見ですが、Trello より断然いいです。笑
そんな Ora について、今回は簡単に紹介します。

## Ora とは

Codemotion 社による、タスクマネジメントツールです。

* [Ora Home](https://ora.pm/home)

こちらのページにもあるように、以下の機能を用意してくれています。(Google 翻訳使用)
<img width="371" alt="スクリーンショット 2019-12-02 23.33.57.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/804cd13c-b7c6-c82f-e86c-05043f292a32.png">

この中でも、良かったと思う機能を紹介していきます。

## 看板・リスト表記

Trello のような看板表記と、タスク一覧をリスト表記する２通りの表記方法があります。デフォルトは看板表記です。（リスト表記をデフォルトにも設定できます）

* 看板表記
<img width="1440" alt="スクリーンショット 2019-12-02 23.39.40.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/2d2e48e5-d76c-b61f-1e9d-ae957934e608.png">

* リスト表記
<img width="1440" alt="スクリーンショット 2019-12-02 23.40.48.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/676aa908-6598-c6b2-6f29-94e82f9ab33b.png">

UI もなかなか素敵ですね。

## タイムトラッキング

これは結構いいなと思った機能で、取り掛かるタスクのタイムトラッキングができます。
操作自体は簡単で、取り掛かる際にスタートを押して、終わったらストップするだけです。
もう少し、砕いて説明すると

① タスクに対して、どのくらいの作業工数がかかるのかを見積もります。
<img width="846" alt="スクリーンショット 2019-12-02 23.52.22.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/835df5b9-51b6-ec66-f5d4-721948608986.png">

② 実際にそのタスクに取り掛かる際に、スタートを押下します。
<img width="827" alt="スクリーンショット 2019-12-02 23.52.43.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/9645f08f-daf5-053f-906b-6e9b77c74542.png">

スタートすると計測が始まり、若干の精神的苦痛を与え始めます。
<img width="808" alt="スクリーンショット 2019-12-02 23.53.04.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/582c5cad-86c3-6012-f3c1-aad5c3aabb2f.png">

ありがたいことに、カードを閉じても計測時間は可視化されます。
<img width="980" alt="スクリーンショット 2019-12-02 23.53.25.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/49b87338-b183-ffbb-9db5-0940a43db017.png">

③ 作業が完了したら、ストップを押下します。
<img width="821" alt="スクリーンショット 2019-12-02 23.54.02.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/1234fac2-805d-b10a-2d6f-5971ee64b77c.png">

以上で、タイムトラッキング作業は完了となりますが、スタート・ストップを押下するという作業を定着させることが必要ですね。（結構ド忘れすることがありそう）
また、このような計測結果はダッシュボードとしても参照することができます。
<img width="1440" alt="スクリーンショット 2019-12-02 23.54.38.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/776ebb31-1d8e-09e6-0f14-646cf0f0a1c1.png">

これは、いいアジャイル開発の運用ができそうな感じがします。
また、こうゆうダッシュボードがあるおかげで、どのタスクを誰にふるかを正しく行うことができるとも思われます。

## チャット

各タスクにおいて、チャットができます。
<img width="800" alt="スクリーンショット 2019-12-03 1.02.05.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/9a66057f-218f-5a21-6101-264193fbd45d.png">

## カレンダーやガントチャート

カレンダーに関しては、結構どのツールにもあるかもしれませんが、もちろん Ora にもあります。
<img width="1440" alt="スクリーンショット 2019-12-03 0.19.38.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/f546eea7-3ffb-f6b6-9ef5-92aba1960c49.png">

さらに、ガントチャートまであるという！！
チームミーティングでは、このガントチャートを参照しながらやれそうですね。
<img width="1440" alt="スクリーンショット 2019-12-03 0.20.08.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/98ac3d86-8aae-e6f0-f53c-4b4add53ab41.png">

さらにさらに、このガントチャート上からタスクの日程を調節でき、分割もすることもできます。
<img width="1439" alt="スクリーンショット 2019-12-03 0.20.42.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/4538dd1c-0008-998b-de7f-b6fcf2f4528e.png">

## Git や Slack など、外部との連携

プロジェクトの設定から、連携したい外部 API を選択すると連携することができます。
<img width="1440" alt="スクリーンショット 2019-12-03 0.26.53.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/60fe87f3-0864-b727-0cc0-faca094feb5f.png">

自分は、 GitHub と Slack で連携してみました。

### GitHub

GitHub のプロジェクトと Ora のプロジェクトを紐づければ、 diff を Ora で参照することができます。
<img width="1440" alt="スクリーンショット 2019-12-03 0.31.48.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/0ffd624e-4993-f726-54f5-97b1865ff220.png">

また、まだ試してませんが、タスクID（ `#1` のような）をコミットログに仕込ませると Ora 側が自動で更新されるようです。（聞いただけでほんとかはまだわかりませんが）

```sh
$ git commit -m "リファクタリングを実施 #1"
```

こんな感じでコミットログを打つと、連携されると信じて今度やってみるつもりです。（←）

### Slack

Ora でのプロジェクトにおいて、ステータスが変更されたり、新たにタスクが追加されたりすると、Slack にその作業が投稿されます。
<img width="1077" alt="スクリーンショット 2019-12-03 0.37.39.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/341734/00971f64-489c-7534-1a3a-23352b9f70c4.png">

いやー、便利です。

## まとめ

UI に関しても、機能に関してもいいところがいっぱいなので、今後とも使用していこうと考えています。
Ora はまだ4年くらい？のツールでもあり、まだ若いですが十分使えると思います。ただ、チーム人数は３人まで無料らしいので、大人数だと課金しないといけない感じです。
まだまだ発見できてない機能とか今後とも探していこうと思います。
気になる方は是非使ってみてください！