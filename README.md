# GPT3_reading_memo

使用図表は、全て論文内で引用されたものである。

# Result

<img width="578" alt="スクリーンショット 2021-03-26 19 22 29" src="https://user-images.githubusercontent.com/11864345/112617709-a6233400-8e68-11eb-90b3-0518ad33e829.png">

petaflop/s-day: １秒間に1peta(10\*\*15)回のニューラルネット上での計算を行うものを１日回した場合の計算量のこと。約10\*\*20回の計算がされる
https://openai.com/blog/ai-and-compute/#fn2

グラフ内の少パラメータの設定6つは、Table2.1で示されていない設定。
クロスエントロピー損失を用いた性能評価では、学習における計算数に対してべき乗則の傾向が見られる（両対数グラフでは直線になる）

## 3.1 Language Modeling, Cloze, and Completion Tasks

### 3.1.1 Language Modeling
言語モデルの評価をPenn Tree Bankデータセットを用いて検証（学習データに含まれるテキストを除外するため、Wikipediaに関係する4つのタスクを除外し、さらに大部分が含まれる10億単語も除外）
GPT-3のzero-shotパラメータ数最大モデルにおいてperplexityで評価を行い、SOTA（GPT-2）より15ポイント改善（35.76 -> 20.50）

### 3.1.2 LAMBADA

<img width="793" alt="スクリーンショット 2021-03-30 13 06 00" src="https://user-images.githubusercontent.com/11864345/112937619-ec2b1100-9162-11eb-9639-bf28998ff88b.png">

文脈を読むことで文の最後の単語を予測するタスクデータセット。
言語モデルのスケーリングによって、このような難しいタスクにおいて収穫逓減（モデルが大きくなっても、性能があまり良くならない現象）が見られる（SOTAのモデルサイズを2倍にしても、わずか1.5%のみ改善したという研究報告もある）
GPT-3のzero-shotでSOTAより8ポイント改善。
one-shot, few-shotでは、以下のような形式を使用し、few-shotにおいてaccuracyを大きく改善した一方、one-shotではzero-shotよりも悪化。これは、one-shotでは出題形式をうまく認識できなかったためと思われる。

### 3.1.3 HellaSwag
物語や手順の終わりとして、最もふさわしい文を選択するデータセット。human accuracyは95.6%だが、言語モデルが解く場合スコアは大きく下がる傾向がある。
SOTAより正解率が低い一方、ファインチューンされたパラメータ数1.5Bの先行研究の言語モデルの正解率は75.4%であり、こちらは上回っている。

### 3.1.4 StoryCloze



## 3.2 Closed Book Question Answering

## 3.3 Translation

GPT-2において英語以外の言語を含むドキュメントはフィルター除外されていたが、英仏間の翻訳において多言語表現の可能性が示唆された（フランス語データは10MB）。
GPT-3では、言語に基づくフィルタリングを実行しない。単語単位では、93%が英語であった（詳細は以下のリンクに記載）。

https://github.com/openai/gpt-3/blob/master/dataset_statistics/languages_by_word_count.csv

比較手法（back-translation）: target言語の文を単言語コーパスから取得し、これをsource言語に翻訳した文を翻訳対とすることで、単言語コーパスから対訳コーパスを疑似的に作成する手法
GPT-3では、言語対を別々に考えずに言語が混在したデータを使用可能

実験結果

<img width="581" alt="スクリーンショット 2021-03-26 18 57 43" src="https://user-images.githubusercontent.com/11864345/112615056-5f800a80-8e65-11eb-889f-4fcde6a985f1.png">

英語への翻訳の場合サンプルを与えるほど性能が高くなる（unsupervised手法ではSOTA）一方、英語からの翻訳の場合あまり高くならない
特にEn-Ro（英語からロシア語）翻訳ではBLEU値が低いが、これはBPEの分割を英語でフィルタリングしたGPT-2で作成したものを使用したため、英語と言語的に似ていないロシア語ではうまく機能しなかった可能性がある。（実験したフランス語やドイツ語は比較的似ている）
付録Hに詳細な実験結果が記載
（個人的な感想としては、パラメータ数が多いほどBLEU値が高くなることは直感と一致するが、zero-shotの場合そうとは限らないことが気になった。）

## 3.4 Winograd-Style Tasks

