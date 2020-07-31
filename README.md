# MedNER-J
## 概要
<!-- 日本語の病名抽出器である[MedEX/J](http://sociocom.jp/~data/2017-MEDEX/index.html)の最新バージョンです． -->

日本語の医療テキストから病名を抽出するシステムです．
[MedEX/J](http://sociocom.jp/~data/2017-MEDEX/index.html)の上位バージョンに相当します．

## 手法
Bidirectional Encoder Representations from Transformers (BERT)の特徴量を元に，条件付き確率場 (Conditional Random Fields: CRF) で病名の抽出を行っています．

BERTは[東北大学乾・鈴木研究室配布の文字ベースモデル](https://www.nlp.ecei.tohoku.ac.jp/news-release/3284/)を使用しています．

## requirements
- python 3.6.1
- torch==1.4.0
- transformers==2.8.0
- allennlp==0.9.0
- mecab-python3==1.0.1

一括インストールは以下のコマンドで行えます．

`python3 -m pip install -r requirements.txt`

<!-- ## データ
- 学習済みモデルファイル
- 病名正規化用辞書ファイル

のダウンロードが必要です．以下のコマンドでダウンロードを行えます．

`sh download_data.sh` -->

## コマンド
- `-i`：入力ファイル名
- `-o`：出力ファイル名
<!-- - -m：モデル（default: BERT） -->
- `-n`：正規化方法（`dict` or `dnorm`, default: `dict`）
- `-f`：出力フォーマット (`xml` or `json`, default:`xml`)

```
python -m medner_j -i sample.txt -o output.txt -f xml
```

入力ファイルは１行１文のテキストファイルを用意してください．

出力フォーマットとして，XML形式とJSON形式を選択できます．それぞれの出力フォーマットについては「使い方」の出力例をご参照ください．

（注）初回の動作時に，モデルファイルと辞書ファイルのダウンロードが行われます（`~/.cache/MedNERJ`）

入出力に関しては，ファイル名を指定しなければ，標準入力・標準出力を使用します．

## 正規化
辞書による正規化（dict）と機械学習による正規化（dnorm）があります．

### 辞書による正規化
辞書は[万病辞書](http://sociocom.jp/~data/2018-manbyo/index.html)を使用します．
略語の展開や類似語検索などは行いません．

フォーマット例：関節液貯留ー＞かんせつえきちょりゅう;icd=E877;lv=C/freq=高;体液貯留

### 機械学習による正規化
[DNormの日本語実装](https://github.com/sociocom/DNorm-J)を使用します．

フォーマット例：関節液貯留ー＞体液貯留

### （任意の関数による正規化）
スクリプトから使用する場合，任意の呼び出し可能な関数による正規化を行えます．


## コマンドから
### コマンド
`python3 main.py -i sample.txt -o sample_output.txt -f xml`

もしくは

`python3 main.py -f xml < sample.txt > sample_output.txt`

### 入力 (sample.txt)
```
それぞれの関節に関節液貯留は見られなかった
その後、左半身麻痺、ＣＴにて右前側頭葉の出血を認める。
```


### 出力 (sample_output.txt) (xml形式)
```
それぞれの関節に<CN value="かんせつえきちょりゅう;icd=E877;lv=C/freq=高;体液貯留">関節液貯留</CN>は見られなかった
の後、<C value="ひだりはんしんまひ;icd=G819;lv=A/freq=高;片麻痺">左半身麻痺</C>、ＣＴにて右前側頭葉の<C value="しゅっけつ;icd=R58;lv=S/freq=高;出血">出血</C>を認める。
```

### 出力 (sample_output.txt) (json形式)
```
[{"span": [8, 13], "type": "CN", "disease": "関節液貯留", "norm": "かんせつえきちょりゅう;icd=E877;lv=C/freq=高;体液貯留"}]
[{"span": [4, 9], "type": "C", "disease": "左半身麻痺", "norm": "ひだりはんしんまひ;icd=G819;lv=A/freq=高;片麻痺"}, {"span": [20, 22], "type": "C", "disease": "出血", "norm": "しゅっけつ;icd=R58;lv=S/freq=高;出血"}]
```

## スクリプトから
```
from medner_j import Ner

sents = [
  "一方で、本邦では体交時の合併症への懸念、マンパワーの不足により腹臥位療法の実施が困難な場合が多い。",
  "腹臥位以外の時間は両側への完全側臥位を実施した。"
  ]

model = Ner.from_pretrained(normalizer="dict")
results = model.predict(sents)
print(results)
```

### 自作正規化関数
```
from medner_j import Ner

class UserNormalizer(object):
    def __init__(self, normalize_dic):
        self.normalize_dic = normalize_dic
    
    def normalize(self, word):
        return self.normalize_dic.get(word)
   
model = Ner.from_pretrained(normalizer=UserNormalizer.normalize)
```

例えば上記のような関数をnormalizerに渡すことで，任意の正規化を行えます


## 開発

- 開発者は仮想環境内で `install.sh` を実行してください
- `fresh_install.sh` を使うとモデルファイルなどのキャッシュも削除します
