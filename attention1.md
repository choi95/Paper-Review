---
layout: post
title:  "Attention[1] Neural Machine Translation by Jointly Learning to Align and Translate(2014) - Review"
date:   2018-02-23 05:00:00 +0900
categories: [deeplearning, rnn, lstm, attention, nlp, paperreview]
---

## 1. Abstract
- Encoder–Decoder Model
    - 이전의 MT(Machine Translation) 분야에서 사용
    - source sentence를 encoding fixed sized vector로 만들고, 이것을 decoding하여 가 번역문을 생성
- **Attention**
    - **실제로 언어에서는 target word마다 더 중요한 source sentence의 정보가 다르다.**
    - **즉, target word에 관련있는 source sentence의 정보를 추가해줘야함**
    - **따라서, target word와 더 관련있는 source sentence의 부분을 참조하도록 모델링**
    - 올바르게 번역될 확률을 높이는 방향으로 학습

-----

## 2. Introduction
- 이전에 MT task 모델에서 주로 사용되던 것은 Encoder–Decoder 모델이다.
    - Encoder : source sentence를 fixed size vector로 요약
    - Decoder : encoding된 vector를 decoding하며 번역
    - **문장이 길어지면, 정보를 압축하는 것이 어려워진다. 그리고 그 과정에서 정보손실이 발생**

- **Attention**
    - 이 문제를 해결하기 위해 Align을 사용하여 Encoder-Decoder 모델을 확장
    - **현재 generating 되는 문장이랑 가장 관련도가 높은 source sentence의 부분들을 찾아냄**
    - Decoder는 위의 방식으로 생성된 context vector와 이전에 생성된 vector를 가지고 generation

-----

## 3. Background : Neural Machine Translation
- 확률적 관점에서의 translation
    - 주어진 source sentence $$x$$에서 target sentence $$y$$가 뽑힐 조건부확률을 최대화 하는 것
    - $${argmax_{y}}(p(y \mid x))$$ .
    - 이를 통해 conditional distribution(softmax의 결과로 나온 확률값들의 분포)를 학습, 즉, target인 $$y$$를 구성하는 $$  y_{1}, y_{2}, \cdots  $$들의 확률값들이 정답과 가장 유사하도록 학습
    
    
# 3.1 RNN Encoder-Decoder
### 3.1.1 Encoder
- source vector $$x = ({x_{1}}, {x_{2}}, \cdots , {x_{Tx}})$$, target vector $$y = ({y_{1}}, {y_{2}}, \cdots, {y_{Ty}})$$
- $$h_{t}$$ = $$f({x_{t}}, {h_{t-1}})$$
    - $$h_{t}$$ = hidden state of Encoder at t-th time step

- $$c$$ = $$q( {h_{1}}, {h_{2}}, \cdots , {h_{Tx}} )$$
    - $$f, q$$ = nonlinear or LSTM
    - input context가 요약된 vector를 의미

### 3.1.2 Decoder
- $$p(y)$$ = $$\prod_{t=1}^{Ty} p( {y_{t}} \mid  {y_{1}}, {y_{2}}, \cdots, {y_{t-1}}, c )$$
    - target sentence $$y$$가 나올 확률은 $$ y_{1}, y_{2}, \cdots, y_{Ty}$$ 가 나올 확률의 곱
    - $$p( {y_{t}} \mid  {y_{1}}, {y_{2}}, \cdots, {y_{t-1}}, c )$$ = $$g( {y_{t-1}}, {s_{t}}, c )$$
    - $$g$$ = nonlinear,  $${s_{t}}$$ = hidden state of Decoder at t-th time step

-----

## 4. Learning to Align and Translate
![decoder](https://files.slack.com/files-pri/T1J7SCHU7-F9DFB9GJ0/m1.png?pub_secret=2ab8d9a87a)
- 기존의 Encoder와 Decoder를 변형하여 새로운 architecture 형성

# 4.1 Decoder: General Description
- 계속해서 attention을 계산
    - previous state를 통해 previous attention이 반영이 되는 구조

- $$p( {y_{t}} \mid  {y_{1}}, {y_{2}}, \cdots, {y_{t-1}}, x )$$ = $$g( {y_{t-1}}, {s_{t}}, {c_{t}} )$$
    - $${s_{t}}$$ = $$f({s_{t-1}}, {y_{t-1}}, {c_{t}})$$ 
    - $${y_{t}}$$가 생성될 조건부 확률을 구하는 방식이 달라짐
    - **$$c$$가 아닌 $${c_{t}}$$를 사용하는데, $${c_{t}}$$는 t-th time step에서 generating 하려는 target과 관련있는 source sentence의 context를 의미**

- annotations = $$( h_{1}, h_{2}, \cdots, h_{Tx} )$$
    - 각각은 t번째 input을 넣어줬을 때, 생성되는 encoder의 hidden state를 의미
    - Encoder를 BRNN으로 할 경우 forward, backward hidden state가 나오므로, 이것을 concatenate하여 생성
    - Encoder를 BRNN으로 구성하게 되면, t번째 단어 주변의 정보를 잘 담게 됨


- $${e_{tj}}$$ = $$a({s_{t-1}}, {h_{j}})$$ where $$a$$ is trainable Neural network   
    - expected annotation
    - previous state와 current annotation의 neural network을 통해서 얻음
    - 실제로 t번째 target word와 j번째 source sentence word의 연관성을 보여줄 annotation

- $$\alpha_{tj}$$ = $$\frac{exp({e_{tj}})}{ {\sum_{k=1}^{Tx} exp(e_{tk})} }$$
    - expected annotation softmax를 취해서 normalization
    - **실제로 t번째 target word와 j번째 source sentence word의 연관성을 가질 확률**
    - 연관성이 큰 context word는 1에 가까원지므로 영향력을 가짐

- $${c_{t}}$$ = $${\sum_{i=1}^{Tx}} \alpha_{ti} {h_{i}}$$
    - annotation들의 가중합이므로, 중요한 부분의 annotation $$h_{i}$$들만 살아남음
    - 따라서, 현재 target에 영향을 미치는 source sequence의 부분을 알 수 있음 
    - **target에 중요한 정보가 선택되어 반영됨**


# 4.2 Encoder: BRNN for Annotating Sequence
- BRNN 구조를 통해서 주변 문맥을 파악
- forward RNN, backward RNN을 concatenate하여 사용

-----

## 5. Experiment Settings
# 5.1 Dataset
- WMT’14
    - Europarl(61M)
    - news commentary(5.5M)
    - UN(421M)
    - others(90M, 272.5M)

# 5.2 Models
- RNN Encoder-Decoder
- RNNsearch (논문에서 제시된 attention model)
- 학습시키는 data의 최소 길이가 30자, 50자 이상인 모델을 각각 구현 

-----

## 6. Result
# 6.1 Quantitative Results
![r3](https://files.slack.com/files-pri/T1J7SCHU7-F9EBHGAVC/r3.png?pub_secret=e782a544b5)
![r1](https://files.slack.com/files-pri/T1J7SCHU7-F9DFBA760/r1.png?pub_secret=7abcbc0136)
- 기존의 모델보다 성능이 좋음을 알 수 있다.
- conventional phrase-based translation system (Moses)보다도 성능이 좋았다.

- input sentence의 길이가 길어져도 좋은 성능을 보인다.

# 6.2 Qualitative Analysis
### 6.2.1 Alignment
![r2](https://files.slack.com/files-pri/T1J7SCHU7-F9CV55DL1/r2.png?pub_secret=ec0766b884)
- 어떤 단어가 연관성을 가지는지 visualization

### 6.2.2 Long Sequence
![r4](https://files.slack.com/files-pri/T1J7SCHU7-F9DH6SHJ6/r4.png?pub_secret=dac9990c0d)

-----

## 7. Related Work
- Learning to Align
- Neural Networks for Machine Translation

-----

## 8. Conclusion
**기존 Encoder-Decoder 모델처럼 전체 sentence를 fixed size vector로 나타내는 것은 정보를 손실한다는 관점에서 비효율적이며, 문장의
 길이가 길어지면 손실되는 정보가 더 많아지기 때문에 성능이 저하된다. 따라서 Attention Model에서는 모델이 input data를 search하여 generating하려는 target과 관련이 있는 부분을 추출하므로 해당 정보에 더 지중하게 해준다. 그러므로 번역을 할 때 정보를 더 많이 제공 받을 수 있으며, 문장의 길이에 상관없이 좋은 성능을 가지게 된다.**

-----

## 9. Reference 
- [https://arxiv.org/abs/1409.0473](https://arxiv.org/abs/1409.0473)
- [http://dalpo0814.tistory.com/45](http://dalpo0814.tistory.com/45)
- [https://ratsgo.github.io/from%20frequency%20to%20semantics/2017/10/06/attention/](https://ratsgo.github.io/from%20frequency%20to%20semantics/2017/10/06/attention/)
- Ybigta deepNLP-study
