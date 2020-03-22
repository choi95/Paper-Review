---
layout: post
title:  "Seq2Seq[1] Learning Phrase Representations using RNN Encoder–Decoder for Statistical Machine Translation(2014) - Review"
date:   2018-02-25 07:00 +0900
categories: [deeplearning, rnn, lstm, seq2seq, nlp, paperreview]
---

## 1. Abstract
- 2개의 RNN으로 Encoder-Decoder 라는 arhcitecture를 제시
    - **Encoder : sequence of symbol을 fixed-size vector로 만들어줌**
    - **Decdoer : fixed-size vector를 사용하여 또 다른 sequence of symbol을 만듦**
    - **source sequence가 주어졌을 때, target sequence를 출력할 조건부 확률를 최대화하는 방향으로 학습**

-----

## 2. Introduction
- 그동안 phrase table로 잡아내지 못하던 linguistic regularity를 극복하였다.

-----

## 3. RNN Encoder-Decoder
# 3.1 Preliminary: Recurrent Neural Networks
- $$h_{t}$$=$$f(h_{t-1}, x_{t})$$
    - previous hidden state와 current input을 이용해 current hidden state를 계산
- $$p(x_{t,j} \mid x_{1}, \cdots , x_{t-1})$$ = $$\frac{exp(w_{j} h_{t})}{\sum_{j'=1}^{K} exp(w_{j'} h_{t}) }$$
    - 현재 hidden state에서 원하는 symbol $$j$$ 가 나올 확률을 구함
    - $$w_{j}$$ = weight matrix $$W$$의 $$j$$번째 column
    - $$K$$ = output이 될 수 있는 모든 symbol의 수(word, character 등등)
- $$p(x)$$=$$\prod_{t=1}^{T} p(x_{t} \mid x_{1}, \cdots , x_{t-1})$$
    - 원하는 target sequence가 나올 확률은 해당 구성요소들이 나올 조건부 확률들의 곱으로 표현됨
    - $$p(x)$$를 최대화하는 방향으로 학습
    
# 3.2 RNN Encoder–Decoder
![ed](https://files.slack.com/files-pri/T1J7SCHU7-F9E84Q2UB/ed.png?pub_secret=ecd1ffd870)
- $$p(y_{1}, \cdots , y_{T'} \mid x_{1}, \cdots, x_{T})$$를 예측하는 것
    - 길이가 다른 sequence를 학습

- Encoder
    - $$h_{t}$$=$$f(h_{t-1}, x_{t})$$ 
    - input symbol을 읽으며 hidden state를 update
    - Encoder RNN의 마지막 hidden state는 input sequence의 summary $$c$$가 됨
- Decoder
    - $$h_{t}$$=$$f(h_{t-1}, x_{t}, c)$$
    - summary $$c$$도 같이 input으로 받음

- $$P(y_{t}, \mid , y_{1} \cdots y_{t-1}, c)$$ = $$g(h_{t}, y_{t-1}, c)$$
    - t-th time step에서 추출될 output의 조건부확률을 구해줌

- $$max_{\theta} \frac{1}{N} \sum_{n=1}^{N} log p_{\theta} (y_{n} \mid x_{n}) $$ .
    - $$\theta$$ = parameters
    - $$(x_{n}, y_{n})$$ = (input seq, output seq) pair of training set
    - conditional log-likelihood를 최대화하는 방향으로 학습
    - decoder의 결과는 미분가능하므로 gradient-based algoritm으로 최적화

# 3.3 Hidden Unit that Adaptively Remembers and Forgets(GRU)
![hidden](https://files.slack.com/files-pri/T1J7SCHU7-F9EB6QEHY/h1.png?pub_secret=644dd49915)
- LSTM을 응용하여 계산과 구현을 쉽게 만든 새로운 unit

- reset gate
    - $$r_{j}$$ = $$\sigma ([W_{r} x]_{j} + [U_{r} h_{t-1}]_{j})$$
    - $$\sigma$$ = sigmoid, $$x$$ = input, $$h_{t-1}$$ = previous hidden step
    - $$j$$ = j-th hidden unit, 즉, input $$x$$의 $$j$$번째 element를 의미
    - 새로운 input을 고려했을 때, previous hidden state를 얼마나 잊을지를 결정

- update gate
    - $$z_{j}$$ = $$\sigma ([W_{z} x]_{j} + [U_{z} h_{t-1}]_{j})$$
    - 새로운 정보를 previous hidden state에 얼마나 반영하여 current hidden state를 생성할지 결정

- hidden state
    - $$h_{j}^{t}$$ = $$z_{j} h_{j}^{t-1} + (1-z_{j}) \widetilde{h_{j}^{t}}$$
    - $$\widetilde{h_{j}^{t}}$$ = $$\phi ([Wx]_{j} + [U(r \odot h_{t-1})]_{j})$$
    - $$\widetilde{h_{j}^{t}}$$는 LSTM의 input gate와 forget gate가 reset gate를 통해 합쳐져있는 구조이다.
    - $$h_{j}^{t}$$는 기존의 cell state와 hidden state가 update gate를 통해 합쳐져있는 구조이다.
    - reset gate를 통해서 previous hidden state를 어느정도 지우고, 그 결과와 update gate를 통해서 current hidden state 생성

-----

## 4. Statistical Machine Translation(SMT)
- 원래는 어떤 단어가 어느 단어로 번역이 되고, 어떤 문법을 통해서 문장이 구성되는지를 모두 모델에 입력했지만, SMT에서는 통계적으로 접근
- 단어마다 번역을 해서 조합하는 방식, 언어학자가 없어도 번역이 가능하다는 장점
- SMT의 목표는 $$p(f \mid e) \propto p(e \mid f)p(f)$$를 maximize하는 것
    - $$p(e \mid f)$$ = translation model, $$p(f)$$ = language model
- 대다수의 SMT 모델은 log linear model $$log p(f \mid e) = \sum_{n=1}^{N} w_{n} f_{n}(f,e) + log Z(e)$$를 사용
    - $$f_{n}, w_{n}$$ = n-th feature, weight, $$Z$$ = 정규화항
- phrase-based SMT
    - translation model $$log p(e \mid f)$$를 각각의 source/target phrase가 올바르게 matching 되었을 확률들로 분해하여 계산
    
- 평가척도로는 BLEU score를 많이 사용

# 4.1 Scoring Phrase Pairs with RNN Encoder–Decoder
- corpus의 단어 빈도수를 무시
    - phrase table로 연산량이 많아지기 때문
    - Encoder-Decoder는 빈도수에 영향을 받지 않으리라고 가정함
- 다양한 올바른 번역을 학습할 수 있는 것을 목표로 함
- 모델이 학습되면 phrase pair를 table에 추가
    - 연산량을 줄이기 위하여

# 4.2 Related Approaches: Neural Networks in Machine Translation(NMT)
Encoder-Decoder 구조를 NMT에 적용가능하다.

-----

## 5. Experiments
# 5.1 Data and Baseline System
- WMT'14 사용
- 모든 데이터를 사용하지 않고 선별적으로 선택하여 학습
- MERT Algorithm으로 weight tunning
- baseline phrase-based SMT : Moses

### 5.1.1 RNN Encoder–Decoder
- word embedding dimension = 100
- weight initializaion by Gaussian Distribution 

### 5.1.2 Neural Language Model
- CSML 모델도 추가적으로 사용

# 5.2 Quantitative Analysis
![result0](https://files.slack.com/files-pri/T1J7SCHU7-F9FD9V72B/r0.png?pub_secret=5836a22de7)

# 5.3 Qualitative Analysis
- ![result1](https://files.slack.com/files-pri/T1J7SCHU7-F9DMX6D5W/r1.png?pub_secret=f8032f3452)
- ![result2](https://files.slack.com/files-pri/T1J7SCHU7-F9EB6GY2G/r222.png?pub_secret=a8fa3ccab2)

# 5.4 Word and Phrase Representations
![result3](https://files.slack.com/files-pri/T1J7SCHU7-F9F7ACANS/r3.png?pub_secret=cb6dbebd62)

-----

## 6. Conclusion
- 다양한 길이의 문장을 다룰 수 있다.
- 서로 다른 문장의 pair를 잘 연결시켰다.
- NMT에 적용하면 더 좋은 효과를 낼 것이다.

-----

## 7. Reference
- [https://arxiv.org/abs/1406.1078](https://arxiv.org/abs/1406.1078)
- [https://medium.com/@jongdae.lim/%EA%B8%B0%EA%B3%84-%ED%95%99%EC%8A%B5-machine-learning-%EC%9D%80-%EC%A6%90%EA%B2%81%EB%8B%A4-part-5-83b7a44b797a](https://medium.com/@jongdae.lim/%EA%B8%B0%EA%B3%84-%ED%95%99%EC%8A%B5-machine-learning-%EC%9D%80-%EC%A6%90%EA%B2%81%EB%8B%A4-part-5-83b7a44b797a)
- [https://jamiekang.github.io/2017/04/23/learning-phrase-representations-using-rnn-encoder-decoder/](https://jamiekang.github.io/2017/04/23/learning-phrase-representations-using-rnn-encoder-decoder/)
