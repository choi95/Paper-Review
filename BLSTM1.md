---
layout: post
title:  "BLSTM[1] Bi-directional LSTM Recurrent Neural Network for Chinese Word Segmentation(2016) - Review"
date:   2018-02-21 06:38:00 +0900
categories: [deeplearning, rnn, lstm, nlp, paperreview]
---

## 1. Abstruct
- RNN은 sequential data에 적합한 Neural Network 구조
- Bi-directional RNN의 cell에 LSTM을 적용하여 BLSTM을 구성
- BLSTM을 사용하여 Chinese word segmentation 작업 수행

-----

## 2. Introduction
- LSTM을 통하여 RNN구조의 memory 능력이 향상되었음
- BLSTM은 과거의 정보뿐만 아니라 미래의 정보에 대한 의존성도 고려할 수 있음

-----

## 3. BLSTM network architecture
# 3.1 LSTM unit
![lstm](https://files.slack.com/files-pri/T1J7SCHU7-F9C0J0T8T/1.png?pub_secret=7ff6b53549)
- [LSTM](https://hwkim94.github.io/rnn/deeplearning/paperreview/2018/02/21/LSTM1.html)은 여러 gate를 통해 정보를 전달하며, 오래된 정보를 기억하는 것에 유리하다.

# 3.2 BLSTM network
![BLSTM](https://files.slack.com/files-pri/T1J7SCHU7-F9D501NQP/2.png?pub_secret=e5fd4fb6a4)
- BLSTM은 sequential dataset에서 과거와 미래 모두의 context를 고려할 수 있다.
- 두 방향으로 진행되는 LSTM cell로 구성
- $${h_{ft}}$$ = forward LSTM의 output($${c_{t}}$$)
    - 일반적인 LSTM의 output과 같음
- $${h_{gt}}$$ = backward LSTM의 output
    - input sequence를 반대방향으로 LSTM에 넣어줌
- $${y_{t}} = [{h_{ft}}, {h_{gt}}]$$ 로 concatenate
    - ![blstm](https://files.slack.com/files-pri/T1J7SCHU7-F9EBK40T0/blstm.png?pub_secret=017b5f1810)
    - 즉, forward LSTM과 backward LSTM에 input으로 $${x_{t}}$$ 가 들어갔을 때의 결과를 concatenate

-----

## 4. Training Method
- 각각의 단어를 labeling하여 segmentaion을 표시
- 각 character를 lookup dictionary를 통하여 dense vector로 만들어 embedding
- BLSTM layer를 여러개 쌓을 경우 parameter가 너무 많아지므로 input vector를 압축
    - 단순히 $${v_{tran}} = {W_{tran}}{v_{t}}$$ 처럼 $$W$$를 곱하여 차원조절
    
-----

## 5. Experiment
![result](https://files.slack.com/files-pri/T1J7SCHU7-F9C2UFKK6/3.png?pub_secret=c92e99f08a)
- 다른 모델보다 성능이 좋았으며, 깊어질수록 성능이 더 좋아졌다.

-----

## 6. Conclusion
- **sequence에서 input data의 위치가 가진 영향력 및 정보 대한 feature를 잘 뽑아낸다.**
- 사전 지식이 없어도 모델링을 할 수 있다.

-----

## 7. Reference
- [https://arxiv.org/abs/1602.04874](https://arxiv.org/abs/1602.04874)
- [http://dalpo0814.tistory.com/43?category=232263](http://dalpo0814.tistory.com/43?category=232263)
