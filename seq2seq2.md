---
layout: post
title:  "Seq2Seq[2] Sequence to Sequence Learning with Neural Networks(2014) - Review"
date:   2018-02-26 07:00 +0900
categories: [deeplearning, rnn, lstm, seq2seq, nlp, paperreview]
---

## 1. Abstract
- sequence의 구조를 가정하지 않는 end-to-end 모델
- [LSTM Encdoer-Decoder](https://hwkim94.github.io/deeplearning/rnn/seq2seq/nlp/paperreview/2018/02/24/seq2seq1.html)
    - **자연어의 특성인, 다양한 길이의 문장을 다룰 수 있음**
- 긴 문장을 다루는 것에도 문제가 없음
- 순서가 중요한 sequence, invariant voice word에 대해서도 robust
- **문장을 거꾸로 넣은 것이 더 성능이 좋았다.**
    - input sequence의 순서만 바꿈

-----

## 2. Introduction
기존의 신경망은 input과 output의 dimension이 정해져있어서 길이가 다양한 sequence를 제대로 담아내지 못했다. 따라서 이 논문에서는
 LSTM [Encoder-Decoder](https://hwkim94.github.io/deeplearning/rnn/seq2seq/nlp/paperreview/2018/02/24/seq2seq1.html)을 이용하여 길이에 상관없이 input과 output을 만들어내는 것을 목적으로 한다. 결과적으로는, input sequence를 반대로 넣는 것이 성능이 더 좋았으며, LSTM 덕분에 긴 문장에 대해서도 성능이 좋았다. 또한, 번역은 source sentence의 의역을 나타내는 경우가 많은데, 이러한 부분도 잘 학습하였다.

-----

## 3. Model
# 3.1 RNN구조의 문제점
- $$h_{t}$$ = $$sigmoid(W^{hx} x_{t} + W^{hh} h_{t-1})$$
- $$y_{t}$$ = $$W^{yh} h_{t}$$
- input을 받으면 previous hidden state와 함께 current hidden state가 계산되고, 이것에 weight가 곱해져 output을 만듦
- 즉, input을 받자마자 output을 만들기 때문에, target의 순서와 어떤 source word가 어떤 target word로 matching 되는지 제대로 학습될 수가 없었음 

# 3.2 Seq2Seq
![model](https://files.slack.com/files-pri/T1J7SCHU7-F9E0KBERF/model.png?pub_secret=90698a0d49)
- Encoder-Decoder
    - RNN를 통하여 input sentence를 fixed size vector로 만들어 정보를 압축하고, 다른 RNN으로 output sequence를 만듦
    - RNN은 long-term-dependency 문제가 있기 때문에, LSTM 사용

- 조건부 확률 $$p(y \mid x)$$ = $$\prod_{t=1}^{T'} p(y_{t} \mid v, y_{1}, \cdots , y_{t-1})$$를 계산
    - $$x$$ = input sequence, $$y$$ = output sequence
    - input sequence $$x$$가 주어졌을 때, output sequence $$y$$가 나올 확률을 높이는 방향으로 학습
    - $$p(y \mid x)$$는 각각의 t-th time step에서 $$y_{t}$$가 나올 확률들의 곱으로 표현됨
    - $$p(y_{t} \mid v, y_{1}, \cdots , y_{t-1})$$는 여태까지 output으로 나온 target word와 Encoder로 요약된 input sequence를 고려했을 때 $$y_{t}$$가 나올 확률
- 모든 문장의 끝에 <EOS> token을 추가해서 문장의 끝임을 같이 학습
    - input sequence에서 <EOS>가 나오면, 그때부터 target sequence를 generate
    - output sequence에서도 <EOS>가 나오면 generate 중단
- 실제 모델에서는 학습할 때만 input을 반대로 넣어줬고, 4-layer DLSTM 구조 사용

-----

## 4. Experiments
# 4.1 Dataset Details
- WMT'14
- 160,000개의 frequent word를 input으로 사용하고, 80,000개의 frequent word를 output으로 사용

# 4.2 Decoding and Rescoring
- soruce sentence $$S$$가 주어졌을 때, 올바른 문장 $$T$$를 출력할 log probablity를 최대화하는 방향으로 학습
    - $$\frac{1}{\mid S \mid} \sum_{s=1}^{\mid S \mid} log p(T \mid S)$$ .

- 학습이 끝나면, 가장 가능성이 높은 번역을 찾아냄
    - $$\widehat{T}$$ = $$argmax_{T} (p(T \mid S))$$ 

- Beam Search를 사용하여 문장을 찾아냄
    - 가능성이 높은 $$B$$개의 문장만 남겨두는 것
    - 새로운 input이 들어오면 $$B$$개의 문장으로부터 가능한 문장들을 다시 계산하고, 다시 가능성이 높은 $$B$$개만 남김

# 4.3 Reversing the Source Sentence
- 문장을 뒤집어서 넣었을 때가 훨씬 성능이 좋았지만, 그 이유에 대한 명확한 설명은 불가능하다.
- minimal time leg 문제로 추정
    - input sentence와 target sentence를 concatenate 시키면 모든 source word가 target word와 멀어지지만, input sentence를 뒤집으면 souce, taget word pair의 평균적인 거리는 비슷하지만, 적어도 introduction부분의 거리는 짧아지므로 학습을 잘 할 수 있었다고 추정

- 반대방향으로 학습하는 것은 긴 문장에 대해서 더 성능이 좋았음
    - 뒤집어서 학습하는 것이 기억부분에서 더 좋은 성능을 보이기 때문이라고 추정


# 4.4 Training Details
- deep LSTM이 shallow LSTM보다 성능이 좋았다.
- [-0.08, 0.08] 범위에서 uniform distribution으로 weight initializaion
- gradient norm의 범위를 [10, 25]로 설정
- 효율성을 위해 비슷한 길이를 가진 문장끼리 batch 형성

# 4.5 Parallelization
8개의 GPU를 사용하여 병렬처리 

# 4.6 Experimental Results
![result1](https://files.slack.com/files-pri/T1J7SCHU7-F9FJCKZ1V/result1.png?pub_secret=caee511e34)

# 4.7 Performance on long sentences
![result3](https://files.slack.com/files-pri/T1J7SCHU7-F9DT09J80/result3.png?pub_secret=91fba1edf2)

# 4.8 Model Analysis
![result2](https://files.slack.com/files-pri/T1J7SCHU7-F9EGTC6EN/result2.png?pub_secret=dcad25ed20)
- 문장의 어순에 따라 위치하는 것이 달라짐, 즉, 문장의 어순을 중요하게 학습

-----

## 5. Related Work
- RNNLM
- NNLM
- LSTM-like RNN
- Encoder-Decoder

-----

## 6. Conclusion
*"Large deep LSTM, that has a limited vocabulary and that makes almost no assumption about problem structure can outperform a standard SMT-based system whose vocabulary is unlimited on a large-scale MT task"*

-----

## 7. Reference
- [https://arxiv.org/abs/1409.3215](https://arxiv.org/abs/1409.3215)
- [https://kangbk0120.github.io/articles/2018-02/seq-2-seq](https://kangbk0120.github.io/articles/2018-02/seq-2-seq)
- Ybigta deepNLP-study
