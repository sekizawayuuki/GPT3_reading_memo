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

このデータセットでは常に文末の単語を予測する設定だが、言語モデルではこの設定は考慮されず次の段落につながる表現が選択される可能性がある。one-shot, few-shotでは、以下のフォーマットで文末単語を予測させる。

<img width="496" alt="スクリーンショット 2021-03-31 17 53 03" src="https://user-images.githubusercontent.com/11864345/113380901-6f917000-93b8-11eb-872d-7a067bda2b79.png">

few-shotにおいてaccuracyを大きく改善した一方、one-shotではzero-shotよりも悪化。これは、one-shotでは出題形式をうまく認識できなかったためと思われる。

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
読解問題タスク（QAタスク）。

<img width="789" alt="スクリーンショット 2021-04-02 13 30 48" src="https://user-images.githubusercontent.com/11864345/113380617-ac109c00-93b7-11eb-89d6-8c837c77b31a.png">

<img width="790" alt="スクリーンショット 2021-04-02 14 20 14" src="https://user-images.githubusercontent.com/11864345/113383281-92bf1e00-93be-11eb-83ab-f34bf9684649.png">

各データセットは解答方法が複数選択や抜き出しなどフォーマットが多岐にわたり、GPT-3は様々な結果となった。SOTAに及ばないが、初期ベースラインや初期結果と同等であった。
自由形式の会話データセットであるCoQAが最もパフォーマンスが高く（human performanceとの差が3ポイント以内）、教師-生徒間の対話から回答を抜き出すデータセットであるQuACが最もパフォーマンスが低かった（ベースラインのELMoよりもF1で13ポイント低い）。 QuACでは構造化された対話行動のモデリングが必要と思われる。
離散的推論や計算能力をテストするデータセットのDROPでは、few-shotでfinetune済BERTをわずかに上回るが、human performanceとは大きく差がある状態。
Wikipediaから抜き出しを行うSQuAD2.0では、zero-shotとfew-shotとの間にF1で10ポイントの改善が見られた。few-shotの結果は元論文のfinetune済の結果をわずかに上回った。
中学(middle)、高校(high)の英語試験の複数選択式データセットであるRACEでは比較的性能が悪く、SOTAより45%劣っており初期モデルと競合する程度であった。

## 3.7 SuperGLUE

SuperGLUEは様々なタスクを含むデータセット（元々GLUEというデータセットがあり、その高難易度版がSuperGLUE）。https://arxiv.org/pdf/1905.00537.pdf

<img width="788" alt="スクリーンショット 2021-04-02 15 58 38" src="https://user-images.githubusercontent.com/11864345/113390269-57c3e700-93cc-11eb-8745-60e1dd897838.png">

<img width="806" alt="スクリーンショット 2021-04-02 15 58 47" src="https://user-images.githubusercontent.com/11864345/113390279-5c889b00-93cc-11eb-9f1d-0b89eb9932a1.png">

few-shotでは全てにおいてK=32を学習データセットからランダムに使用。
COPA（前提と質問と回答候補（2つ）が与えられ、より適切な回答を選択するタスク）, ReCoRD（パラグラフとクエリ（文）が与えられ、クエリ内の空欄をパラグラフから抜き出すタスク）ではSOTAに近いスコアを達成。
WSC（代名詞を含む文において、代名詞と名詞のペアが同じものであるかを判定するタスク）では80.1%と比較的良い結果となった（オリジナルのWinogradでは88.6%）。
BoolQ（テキストと質問が与えられ、Yes/Noの2値で回答するタスク）、MultiRC（テキストといくつかの解答候補が与えられ、正しい回答を全て選択するタスク）、RTE（前提のテキストと仮説の文が与えられ、仮説が前提に含まれているかどうかを回答するタスク）では、finetune済みBERT-Largeとほぼ同等。（ランダム選択が56%）
CB（前提のテキストと仮説の文が与えられ、仮説が前提に対して含意/矛盾/不明（中立）のどれであるかを回答するタスク）では比較手法より劣る結果となった。
WiC（2つの文が与えられ、その両方に出現する名詞が同じ意味かどうかを回答するタスク）では49.4%とかなり低い結果となった。
3.8節で詳細は言及されるが、WiC、RTE、CBの結果から2文を比較するタスクではGPT-3のone-shot, few-shotがうまくいかない傾向がある。

図3.8から、モデルサイズやサンプル数Kが大きくなるほどスコアも高くなる傾向がある。（Kを32より大きくしても改善されなかった）。finetune済BERT-Largeのスコアと比較すると、Kは最低でも8は必要。

## 3.8 NLI

与えられた2文間の関係を予測するタスク。NLI（I: 推論）では、モデルは2番目の文が1番目の文に対して含意、矛盾、中立（中立が選択肢にない場合もある）のどれであるかを分類する。
SuperGLUEのデータセットの1つであるRTEは、含意or矛盾の2値分類タスク。
さらに難しくなったデータセットANLI（A: Adversarial）を用いて評価。R1, R2, R3の３つに分けられている。

<img width="791" alt="スクリーンショット 2021-04-02 17 07 01" src="https://user-images.githubusercontent.com/11864345/113396097-dffaba00-93d5-11eb-822c-982b02d65b59.png">

<img width="1063" alt="スクリーンショット 2021-04-02 17 17 27" src="https://user-images.githubusercontent.com/11864345/113396984-55b35580-93d7-11eb-9967-6713af94916c.png">

付録の結果から、GPT-3よりもパラメータ数が少ない場合はrandom choiceと大きく変わらないが、GPT-3ではR3において大きく上回った。（RTEの結果は図H.1を参照）
NLIタスクは、言語モデルが解くには難しいという状況である。

## 3.9 Synthetic and Qualitative Tasks
モデルの能力の幅を検証するために、追加実験を行った。

### 3.9.1 Arithmetic
自然言語で表された算数の計算を行うタスク。

<img width="859" alt="スクリーンショット 2021-04-02 17 37 04" src="https://user-images.githubusercontent.com/11864345/113398776-12a6b180-93da-11eb-899d-679f63388d1c.png">

<img width="847" alt="スクリーンショット 2021-04-02 17 52 51" src="https://user-images.githubusercontent.com/11864345/113400221-6adeb300-93dc-11eb-8fc1-d04735ecd309.png">


digit: 桁数
addition: 加算 (Q: What is 48 plus 76? A: 124.)
subtraction: 減算 (Q: What is 34 minus 53? A: -19)
multiplication: 乗算 (Q: What is 24 times 42? A: 1008)
Ops(operations): 2つの演算子（+,-,\*のどれか）が含まれる計算 (Q: What is 6+(4\*8)? A: 38)

加算、減算では、2桁加算では100%、減算では98.9%、3桁ではそれぞれ80.2%、94.2%と高い正解率を示し、桁数が多いほど正解率が下がる（一般化する能力はある）。
乗算や複数演算子の正解率は20%台であり、単純ではない計算に対してもある程度対応できている。
zero-shot, one-shotはfew-shotと比較すると正解率が低く、計算を行うタスクへの適応が重要といえる
学習データでの出現を記憶しているだけの可能性があるため、3桁の加算減算2000件に対して"<NUM1> + <NUM2> =" および "<NUM1> plus <NUM2>"のような出現があるかを確認したところ、加算が17件、減算が2件のみであり、ほとんどは出現していなかった。
また、不正解であった回答を確認すると、繰り上がりの1が反映されていなかったなどの間違いが多く、実際に計算しようとしていると思われる。

### 3.9.2 Word Scrambling and Manipulation Tasks
ある英単語に対して文字に関する編集を行い、元の単語を予測するタスク。以下の5つを設定。

• Cycle letters in word (CL): 単語の文字をループさせ始点を変更した場合に、元の単語を予測。(lyinevitab = inevitably)
• Anagrams of all but first and last characters (A1): 最初と最後の1文字ずつ以外が並び替えられている場合に、元の単語を予測。(criroptuon = corruption)
• Anagrams of all but first and last 2 characters (A2): 最初と最後の2文字ずつ以外が並び替えられている場合に、元の単語を予測。(opoepnnt = opponent)
• Random insertion in word (RI): 句読点やスペースが各文字の後ろにランダムに挿入されている場合に、元の単語を予測。(s.u!c/c!e.s s i/o/n = succession)
• Reversed words (RW): 単語の文字が逆に並び替えられている場合に、元の単語を予測。(stcejbo = objects)

5~14文字からなる高頻度単語10,000語に対して実験（論文 `Natural language corpus data` を用いて取得）

<img width="865" alt="スクリーンショット 2021-04-02 18 14 26" src="https://user-images.githubusercontent.com/11864345/113401989-4afcbe80-93df-11eb-9460-262978c3603d.png">

<img width="852" alt="スクリーンショット 2021-04-02 18 18 22" src="https://user-images.githubusercontent.com/11864345/113402350-d6764f80-93df-11eb-878b-0e5e2fc2fe78.png">


モデルサイズが大きいほど正解率が上昇する傾向があり、簡単なタスクCL, RIでは正解率が高めである一方難しいタスクAは低めでありRWはモデルサイズにかかわらず解けない。
zero-shotではほとんどの問題が解けず、one-shotでもfew-shotと比較すると大幅な性能の低下が見られた。testの段階でタスクを学習できていると思われる。（学習データには出現しないと思われるため。）
これらのタスクを解くには文字レベルの操作が必要だが、BPEでは1トークンあたり0.7単語分の分解が行われるため、より細かい分解が必要となる。
また、CL, A1, A2は、元の単語が編集後の単語と1:1対応ではないため、元の単語を予測するために検索のような処理が必要と思われる。

### 3.9.3 SAT Analogies
374のSATアナロジー問題を収集。2つの単語間関係が与えられ、それと同じ関係を持つ別の単語ペアを選択する問題。2005年以前のSAT大学入学試験を使用
例： “audacious is to boldness as (a) sanctimonious is to hypocrisy, (b) anonymous is to identity, (c) remorseful is to misdeed, (d) deleterious is to result, (e) impressionable is to temptation” 正解は(a)。

<img width="856" alt="スクリーンショット 2021-04-02 18 29 25" src="https://user-images.githubusercontent.com/11864345/113403307-6072e800-93e1-11eb-8150-78b7fb9d193d.png">

モデルサイズが大きいほど正解率が高くなる傾向がある。受験者の平均正解率は57％であり、one,few-shotで上回る（ランダムだと20%）

### 3.9.4 News Article Generation
GPT-3が生成したニュース記事と人間が作成した記事を人間が見分けられるかで検証

<img width="841" alt="スクリーンショット 2021-04-23 18 52 36" src="https://user-images.githubusercontent.com/11864345/115854372-21c2d180-a465-11eb-99b5-dfbf5b6180d9.png">

Mean Accuracy が低いほど、人間作成 or 機械作成を見分けられなかったことを表す。

生成例は以下の通り。

<img width="863" alt="スクリーンショット 2021-04-23 18 57 00" src="https://user-images.githubusercontent.com/11864345/115854946-b62d3400-a465-11eb-8ff9-9b4f35fb290d.png">

<img width="854" alt="スクリーンショット 2021-04-23 18 57 18" src="https://user-images.githubusercontent.com/11864345/115854997-c0e7c900-a465-11eb-8a5c-ba13b1c9eb03.png">


### 3.9.5 Learning and Using Novel Words
1回だけ定義された単語を見た後、その単語を使用するタスク（1度使われた単語の意味を予測するタスクもあるが、今回は対象外）。
存在しない単語を含む文とその前の説明文のペアをあらかじめ与え、few-shotのような設定とする。

<img width="775" alt="スクリーンショット 2021-04-02 18 49 34" src="https://user-images.githubusercontent.com/11864345/113405084-2fe07d80-93e4-11eb-8d62-8e745d5ef38e.png">

太字がGPT-3の出力、それ以外は人手で作成された。
GPT-3の生成文は正しいor尤もらしいように見える。

### 3.9.6 Correcting English Grammar
文法訂正を以下のようなフォーマットで行う。（"Poor English Input: <sentence>\n Good English Output: <sentence>"）
  
<img width="642" alt="スクリーンショット 2021-04-02 18 40 10" src="https://user-images.githubusercontent.com/11864345/113404291-e0e61880-93e2-11eb-91ac-2e57dff5887a.png">

太字がGPT-3の出力、それ以外は人手で作成された。
good, poorの意味を文法としての意味であると判定できていない場合がある。例えば下から3番目では、cheapを削除している。
