## ntagger

reference pytorch code for named entity tagging

## requirements

- python >= 3.6

- pip install -r requirements.txt

- pretrained embedding
  - glove
    - [download Glove6B](http://nlp.stanford.edu/data/glove.6B.zip)
  - unzip to 'embeddings' dir
  ```
  $ mkdir embeddings
  $ ls embeddings
  glove.6B.zip
  $ unzip glove.6B.zip 
  ```

- additional requirements for BERT(huggingface's [transformers](https://github.com/huggingface/transformers.git))
```
$ pip install tensorflow-gpu==2.0
$ pip install git+https://github.com/huggingface/transformers.git
```

- data
  - CoNLL 2003 english
    - `data/conll2003`
    - from [etagger](https://github.com/dsindex/etagger)
    - [SOTA on CoNLL 2003 english](https://paperswithcode.com/sota/named-entity-recognition-ner-on-conll-2003)

## CoNLL 2003 english

### emb_class=glove

- train
```
* token_emb_dim in config.json == 300 (ex, glove.6B.300d.txt )
$ python preprocess.py
$ python train.py
* --use_crf for adding crf layer
$ python train.py --use_crf

* tensorboardX
$ rm -rf runs
$ tensorboard --logdir runs/ --port ${port} --bind_all
```

- evaluation
```
$ python evaluate.py
INFO:__main__:[F1] : 0.8569903948772679, 3684
INFO:__main__:[Elapsed Time] : 57432ms, 15.589576547231271ms on average
* seqeval.metrics supports IOB2(BIO) format, so FB1 from conlleval.pl should be similar value with.
$ cd data/conll2003; paste -d ' ' test.txt pred.txt > test-pred.txt ; perl ../../etc/conlleval.pl < test-pred.txt ; cd ../..
accuracy:  96.80%; precision:  86.10%; recall:  85.30%; FB1:  85.70
              LOC: precision:  86.43%; recall:  91.61%; FB1:  88.94  1768
             MISC: precision:  76.11%; recall:  70.80%; FB1:  73.36  653
              ORG: precision:  83.27%; recall:  80.01%; FB1:  81.61  1596
              PER: precision:  92.72%; recall:  90.54%; FB1:  91.61  1579

* --use_crf
$ python evaluate.py --use_crf
INFO:__main__:[F1] : 0.8605357142857142, 3684
INFO:__main__:[Elapsed Time] : 156063ms, 42.362377850162865ms on average
  * token_emb_dim: 100
  INFO:__main__:[F1] : 0.8587449933244327, 3684
  INFO:__main__:[Elapsed Time] : 344884ms, 93.61672095548317ms on average (cpu)
```

- best : **86.05%** (test set)
  - this value is close to 'FB1:  86.48' of [etagger](https://github.com/dsindex/etagger)'s result.
    - using word embedding only.

### emb_class=bert

- train
```
* ignore token_emb_dim in config.json
* n_ctx size should be less than 512
* download 'bert-large-cased' to './'
$ python preprocess.py --emb_class=bert --bert_model_name_or_path=./bert-large-cased

* fine-tuning
$ python train.py --emb_class=bert --bert_model_name_or_path=./bert-large-cased --bert_output_dir=bert-checkpoint --batch_size=16 --lr=1e-5 --epoch=10

* --use_crf for adding crf layer

* --bert_use_feature_based for feature-based

* --bert_disable_lstm for removing lstm layer

* tensorboardX
$ rm -rf runs
$ tensorboard --logdir runs/ --port port-number --bind_all
```

- evaluation
```
$ python evaluate.py --emb_class=bert --bert_output_dir=bert-checkpoint --data_path=data/conll2003/test.txt.fs

$ cd data/conll2003; paste -d ' ' test.txt pred.txt > test-pred.txt ; perl ../../etc/conlleval.pl < test-pred.txt ; cd ../..

* fine-tuning
  * bert-large-cased
    INFO:__main__:[F1] : 0.9113453192808433, 3684
    INFO:__main__:[Elapsed Time] : 170391ms, 46.251628664495115ms on average
    * --use_crf
      1) lstm_dropout:0.0, lr:1e-5
        INFO:__main__:[F1] : 0.908178536843032, 3684
        INFO:__main__:[Elapsed Time] : 244903ms, 66.47747014115092ms on average
        FB1:  91.07 (by conlleval.pl)
      2) lstm_dropout:0.1, lr:1e-5
        INFO:__main__:[F1] : 0.9071403447062961, 3684
        INFO:__main__:[Elapsed Time] : 246333ms, 66.86563517915309ms on average
        FB1:  90.91
      3) lstm_dropout:0.1, lr:2e-5
        INFO:__main__:[F1] : 0.8962568711281739, 3684
        INFO:__main__:[Elapsed Time] : 237240ms, 64.39739413680782ms on average
        FB1:  89.84
    * --bert_disable_lstm
      INFO:__main__:[F1] : 0.9006085192697768, 3684
      INFO:__main__:[Elapsed Time] : 138880ms, 37.69815418023887ms on average
    * --use_crf --bert_disable_lstm
      INFO:__main__:[F1] : 0.9044752682543836, 3684
      INFO:__main__:[Elapsed Time] : 214022ms, 58.09500542888165ms on average
      FB1:  90.65
```

- best : **91.13%** (test set)

## references

- [transformers_examples](https://github.com/dsindex/transformers_examples)

