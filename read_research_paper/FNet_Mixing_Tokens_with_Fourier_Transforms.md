# FNet: Mixing Tokens with Fourier Transforms

## リンク

- https://arxiv.org/abs/2105.03824
- https://github.com/google-research/google-research/tree/master/f_net
- https://github.com/rishikksh20/FNet-pytorch

## 概要

- Transformerのself-attention layerをフーリエ変換に置き換えることで、GLUEのBERTベンチマークでBERT modelの精度の92-97%の精度
  - ベクトルやベクトル列に変換を加えることをmixingと呼んでいる
  - randomな値が入っている行列でやった場合は精度が悪い → ある程度、構造化された変換であることが大事
  - フーリエ変換なので学習パラメータはなし
- 入力サイズ（sequenceサイズ）512で、Transformerモデルと比べてGPU学習では80%速く、TPU学習は70%速い
  - 長い入力サイズ（sequenceサイズ）では、特にFNetの方が速い

## Transformerとか

- https://tips-memo.com/translation-jayalmmar-transformer
- https://deepsquare.jp/2021/05/gmlp/

ここら辺が参考になるかも

## フーリエ変換イメージ

https://ja.wikipedia.org/wiki/%E3%83%95%E3%83%BC%E3%83%AA%E3%82%A8%E5%A4%89%E6%8F%9B のgifがイメージしやすい

## FNetのフーリエモジュール

1. sequence方向に離散フーリエ変換
2. hidden demension方向に離散フーリエ変換

<img src="https://github.com/imyutaro/note/blob/master/read_research_paper/img/FNet_Mixing_Tokens_with_Fourier_Transforms/image-20211103-144617.png" width="50%" />


下図のFourierとなっている部分で上図の変換を行うだけ。

<img src="https://github.com/imyutaro/note/blob/master/read_research_paper/img/FNet_Mixing_Tokens_with_Fourier_Transforms/image-20211103-144912.png" width="50%" />


## 実験

### Transfer Learning

モデル全体の評価のために、実験では以下の5つのモデルでbenchmarkを比較。

- BERT-Base: 普通のTransformerモデル
- FNet encoder: self-attentionをフーリエ変換に置き換えたモデル
- Linear encode: self-attentionを2つのFeedForwardに置き換えたモデル
  - 1つ目のFeedForwardはsequence方向に適用して、2つ目のFeedForwardはhidden dimentionに適用する。この2つのFeedForwardは学習する。
- Random encoder: self-attentionを2つのrandom行列に置き換えたモデル
  - random行列: 適当に乱数で作成した行列
  - 1つ目のrandom行列はsequence方向に適用して、2つ目のrandom行列はhidden dimentionに適用する。この2つのFeedForwardは学習する。
- FeedForward-only（FF-only）encoder: Transformerモデルからself-attentionを除いたモデル（token mixingがないモデル）

### 結果

- FNetもLinear encoderも7.5~8%程、BERT-Baseと比べて精度が悪いだけでそこまで悪くない（Table1参照）
- ランダムは精度がよくない（Table1参照）
  - → ある程度、構造化されたmixingをしないと精度が上がらない

## Long-Range Arena (LRA) benchmark

メモリ効率と学習速度の評価のために、以下の4モデルでbenchmarkを比較。

- Vanilla Transformer: 普通のTransformer
- Performer: Transformer系で一番速いモデル
- FNet: 前述
- Linear encoder: 前述

### 結果

- FNetのフーリエモジュールは学習パラメータがないので他のモデルに比べてメモリを多く使う必要がない（Table5参照）
- 推論時間は入力系列（input sequence）が長いほどFNetが速い（Table6参照）
  - Transformerだと入力系列が8192だとout of memoryで推論ができていない

## Appendices

- 他の変換方法として、離散コサイン変換（DCT）やアダマール変換（Hadamard Transform）を試したが、DFTよりもaccuracyが少し悪かった

## 感想

[Patches Are All You Need?](https://openreview.net/forum?id=TVHS5Y4dNvM) という論文を見て、FNetという論文が関係あるというのを聞いたのでFNetを読みました。「Pathes Are All You Need?」では、self-attensionやMLP-Mixerの構造よりもパッチ化することが大事と主張していますが、そんなことないんじゃないかなぁと自分は思います。FNetで主張していることが大事なことで、隠れ層内のベクトル要素を特定の規則に則って混ぜることが大事なんだと思います。（「Pathes Are All You Need?」では、attentionのような操作をCNNの枠組みで実装しているので、CNNの枠組みでattentionを実現しているところが大事なところだと思います。）

- https://blog.shikoan.com/patches-are-all-you-need/
- https://www.slideshare.net/DeepLearningJP2016/dlpatches-are-all-you-need-convmixer

DFTでは精度がなんで出るのかとかを深ぼっていくと、どのような変換が精度を良くするのかとか、そもそもなぜMLP-Mixierとかが精度出るのかとかの足掛かりになるのかなと思ってます。attentionとかMLP-Mixerは学習パラメータで変換（Mixing）しているので、任意の変換をしているって考えられます。なので、もう少し本質的な、どのような変換がいいのかがわかると面白いなぁと考えてます。

