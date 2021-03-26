# GPT3_reading_memo

# Result

## Translation

GPT-2において英語以外の言語を含むドキュメントはフィルター除外されていたが、英仏間の翻訳において多言語表現の可能性が示唆された（フランス語データは10MB）。
GPT-3では、言語に基づくフィルタリングを実行しない。単語単位では、93%が英語であった（詳細は以下のリンクに記載）。
https://github.com/openai/gpt-3/blob/master/dataset_statistics/languages_by_word_count.csv
比較手法（back-translation）: target言語の文を単言語コーパスから取得し、これをsource言語に翻訳した文を翻訳対とすることで、単言語コーパスから対訳コーパスを疑似的に作成する手法
GPT-3では、言語対を別々に考えずに言語が混在したデータを使用可能

実験結果

