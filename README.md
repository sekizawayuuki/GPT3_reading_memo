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

### 3.1.4 StoryCloze (2016 dataset)
5文からなる物語の最後の文を選択するタスク。
zero-shotでは83.2%、few-shotでは87.7%(K = 70)となりSOTAのBERTモデルには届かず。

## 3.2 Closed Book Question Answering
open-book: 情報検索のモデルを使用し、質問・検索テキストを用いて回答を生成するモデル。
closed-book: 質問文から直接回答文を生成するモデル。こちらの方が情報量が少ないため、制限が大きい（closed-bookのSOTA手法の論文 `How much knowledge can you pack into the parameters
of a language model?` より）

<img width="892" alt="スクリーンショット 2021-04-01 23 09 06" src="https://user-images.githubusercontent.com/11864345/113307150-ca3ab580-933f-11eb-8ed3-87c4b9d1e63a.png">


<img width="892" alt="スクリーンショット 2021-04-01 23 09 06" src="https://user-images.githubusercontent.com/11864345/113306684-4f719a80-933f-11eb-9110-d4b3b8f5b10e.png">

TriviaQAデータセットでは、zero-shotでもclosed-book finetuneモデルを上回り、one-shotではオープンドメインのQAシステムと並び、few-shotでは上回った。
WebQuestionsデータセットでは、few-shotでSOTAに近い結果となった。zero-shotと比べると大きく改善したため、TriviaQAと比較するとWebQuestionsデータセットの質問および/あるいは解答のスタイルがGPT-3の分布に含まれていない可能性がある（few-shotにおいて、スタイル適用ができていると思われる）。
NaturalQuestionsデータセットでは、先行研究の正解率に及ばず、zero,one,few-shot間の変化はWebQuestionsの結果と似ている。これは、データセットで質問される内容がWikipediaの細かい知識を用いる必要があるもののため、モデルの限界である可能性がある。
図3.3に示すように、モデルサイズが大きいほど正解率が上がるため、モデルの容量が知識としてパラメータに吸収されることを示していると言える

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
Winograd Schemas Challenge: 代名詞がどの名詞を指しているかを予測するタスク。文法的には曖昧だが意味を考慮すると解ける。
2012年バージョンのWinogradデータセットではfinetune済言語モデルでhuman performanceに近い性能を示したが、2019年バージョンの（難しくなった）Winograndeデータセットでは差が開いた。
実験ではGPT-2と同じ設定である部分評価で行った

<img width="883" alt="スクリーンショット 2021-04-01 23 12 44" src="https://user-images.githubusercontent.com/11864345/113315986-d8410400-9348-11eb-8bb8-124238ce48b9.png">

Winogradでは、human performanceに近い性能のSOTAに近い結果となった（WinogradスキーマのデータがGPT-3の学習データに少し含まれており論文内ではdata contamination: データ汚染と呼ばれている、詳細はsection4で述べられるが影響はわずかとのこと）。
Winograndeでは、zero,one,few-shotとなるごとに性能が高くなった。他手法との比較としては、fine-tune済RoBERTAモデル: 79%, SOTA(finetune済T5): 84.6%, human performance: 94.0%である。

## 3.5 Common Sense Reasoning
PhysicalQA: 物理系常識のQA
ARC: 3年~9年の科学実験から収集された複数選択式QA。Challengeバージョンは、統計的手法や情報検索手法では正しく回答できない。

<img width="878" alt="スクリーンショット 2021-04-02 0 17 34" src="https://user-images.githubusercontent.com/11864345/113323209-a92e9080-9350-11eb-8cfa-5028cc4cf65a.png">

PhysicalQAでは、zero,one,few-shot全てにおいてSOTA(finetune済RoBEATa)を上回った一方、human performanceを10%以上下回った。
データサイズを大きくしてもそれほど性能は改善せず、加えて（testのラベルは公開されていないが）データ汚染の問題があると思われるため、table3.6に\*を付加（詳細はsection4）。

ARCでは、Challengeバージョンにおいてベースライン(finetune済RoBEATa)の55.9%に近い結果となった。Easyバージョンでは同ベースラインをわずかに上回った。（リーダーボードに記載されているが、67.75%）
OpenBookQAでは、few-shotで性能が改善したが、ベースラインであるfinetune済BEAT Largeと同等であった。（リーダーボードに記載されているが、60.4%）

PIQA, ARCではone-shotでzero-shotをわずかに下回る一貫性のなさが見られたが、OpenBookQAでは大きく改善した。

## 3.6 Reading Comprehension


