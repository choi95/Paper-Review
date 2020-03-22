---
layout: post
title: "Attention[2] Effective Approaches to Attention based Neural Machine Translation(2015) - Review"
date:   2018-02-23 07:00:00 +0900
categories: [deeplearning, rnn, lstm, attention, nlp, paperreview]
---

# 1. Abstract
- 기존의 NMT 모델들과 attention 모델의 성능비교가 목적 
- attention model
    - **source sentence에 선택적으로 집중, 즉, 관계있는 정보에 집중**
- **global approach : source 전체에서 attention**
- **local approach : source의 일부분에서만 attention**

-----

# 2. Introduction
- standard MT
    - phrase table과 language model을 저장하여 사용
    - 복잡한 decoder
- NMT
    - phrase table과 language model 등 사전지식의 저장이 필요 없는 end-to-end로 학습
    - **따라서, 상대적으로 small memory footprint**
    - 문장의 길이에 상관이 없어서, 긴 문장에도 일반화가능
    - 간단한 decoder

- attention
    - **연관이 높은 것을 찾는 모델이므로 modality(양식)에 상관없이 적용가능**
    - ex) speech와 text의 연결, picture와 text
    - 서로다른 언어로 구성된 문장간의 연관성을 연결시켜 번역작업을 진행 가능

- global attention
    - 모든 source word가 attended됨('참조'됨)

- local attention
    - hard attention과 soft attention의 혼합 
    - global attention보다 연산량이 적음
    - diffentiable

-----

# 3. Neural Machine Translation(NMT)
![nmt](https://files.slack.com/files-pri/T1J7SCHU7-F9DL6EN1Z/nmt.png?pub_secret=e5aa493a0d)
- NMT는 $$x_{1}, \cdots , x_{n}$$로부터 $$y_{1}, \cdots , y_{m}$$로 바꿀 조건부확률 $$p(y \mid x)$$ = $$\prod_{i=1}^{m} p(y_{i} \mid y_{<i}, s)$$을 모델링한다. 
    - **i-th time step에 $$y_{i}$$를 만들어낼 확률을 최대화하도록 학습**
    - $$p(y_{i} \mid y_{1}, \cdots , y_{i-1}, s)$$ = $$softmax(g(h_{i}))$$
    - $$h_{i}$$ = $$f(h_{i-1}, s)$$
    - $$g$$ = hidden state를 vocabulary vector로 바꿔주는 function, $$f$$ = RNN, LSTM, GRU 등등 cell funtion
    - $$h$$ = hidden state, $$s$$ = source sentence의 정보가 압축된 것
- Encoder : source sentence로 부터 정보를 압축하여 $$s$$ 만들어냄
- Decoder : $$s$$와 previous state를 통하여 target을 generate
- 보통 NMT에서는 $$s$$를 decoder가 시작할 때만 한 번 사용
    - 이것을 계속 사용해주는 것이 attention model 

-----

# 4. Attention-based Models
- attention mechanism
    - soruce sentence로 부터 i-th time step의 target $$y_{i}$$를 예측하는데 필요한 문맥 정보만 추출
    - 즉, 각 time step마다 source-side context vector $$c_{i}$$를 만드는 것
    - $$\widetilde{h_{t}}$$ = $$tanh(W_{c}[c_{t};h_{t}])$$
    - $$p(y_{i} \mid  y_{1}, \cdots , y_{i-1}, s)$$ = $$softmax(W_{s} \widetilde{h})$$ 를 학습


## 4.1 Global Attention
![global](https://files.slack.com/files-pri/T1J7SCHU7-F9DP3CYSW/global.png?pub_secret=851fef9b08)
- Global attention model 
    - 모든 source sentence 의 hidden state를 고려하여 align vector를 만듦

- global align vector
    - 해당 target에 필요한 soruce sentence의 부분들을 encoding한 것
    - 각 time step마다 t-th time step $$a_{t}$$를 만듦
    - $$a_{t}(s)$$ = $$align(h_{t}, \overline{h_{s}})$$ = $$\frac{exp(score(h_{t}, \overline{h_{s}}))}{\sum_{s'}^{n} exp(score(h_{t}, \overline{h_{s'}}))}$$
    - $$a_{t}(s)$$는 t-th time step에 현재 target과 $$s$$번째 source word간에 얼마나 관계가 있는지를 알려줌
    - ![score](https://files.slack.com/files-pri/T1J7SCHU7-F9F2LK9LN/score.png?pub_secret=eea65c1c13)
    - $$\overline{h_{s}}$$ = source hidden state, $$h_{t}$$ = current target hidden state
    - 즉, 현재 target과 각각의 source sentence의 word를 비교하여 관련있는 것들을 찾아내는 과정

- context vector
    - $$c_{t}$$ = $$\sum_{t=1}^{n} {a_{t} \overline{h_{t}}}$$
    - 각 source word의 hidden state와 align vector를 곱해서 가중합을 구해주므로, 관련도가 높은 source만 추출됨

## 4.2 Local Attention
![local](https://files.slack.com/files-pri/T1J7SCHU7-F9EQNA287/local.png?pub_secret=6f6957e761)
- Global attention의 경우 모든 source word에 대한 align vector를 구해줘야하므로, 비효율적이며 긴 문장을 다룰 때는 비현실적이다.

### 4.2.1 soft & hard attention
- local attention은 soft attention과 hard attention의 단점을 극복한 attention 방식
- soft attention 
    - global attention과 유사
    - 모든 단어에 대하여 'soft'하게 align 계산, 즉, 관계성을 '약하게' 파악
- hard attention
    - 특정 부분에 'hard'하게 align 계산, 즉, 특정부분에 대한 관계성을 '강하게' 파악
    - 본래 개념이 제시되었던 image분야에서는 특정부분을 잘라낸다는 것은 window만큼 자르므로 비연속적이어서 미분불가능
    - ex) 32x32x3 image를 4x4x3 image로 잘라내면 잘린 단면에서 바로 옆에 있던 vector와 절단됨 
- **hard attention은 미분 불가능하다는 단점이 있지만, local attention에서는 window size를 작게 잡아 연속한 context의 일부에만 focus해서 계산비용이 줄어 global attention의 단점을 극복하며 동시에 미분가능하다.**

### 4.2.2 aligned position
- 각 time step마다 aligned position $$p_{t}$$를 generate
    - align vector를 계산할 window의 위치를 결정
    - context vector는 $$[p_{t}-D, p_{t}+D]$$ 범위에서 결정되며, $$D$$는 empirically selected
- global attention에서는 source sentence의 길이에 따라 $$a_{t}$$의 size가 정해졌지만, local attention의 경우 window의 size D가 결정되므로 $$a_{t}$$도 fixed 2D-size vector가 됨

- monotonic alignment 
    - $$p_{t}$$ = $$t$$
    - 점진적으로 window의 size를 뒤로 옮김
    - $$a_{t}$$는 global attention처럼 window size에 있는 각각의 source들의 연관성으로 계산됨

- predictive alignment
    - $$p_{t}$$ = $$S \cdot sigmoid(v_{p}^{T} tanh(W_{p} h_{t}))$$
    - $$W_{P}, v_{p}$$는 모델과 함께 학습되는 parameter, $$S$$ = source length
    - target과 관계가 있는 position을 예측하여 그 부분에서 alignment 계산
    - 따라서 alignment $$a_{t}$$가 $$p_{t}$$주변과 관련이 있을 것이라고 추정되므로, $$p_{t}$$를 중심으로 gaussian distribution을 고려함
    - $$a_{t}$$ =  $$align(h_{t}, \overline{h_{s}}) \times exp(- \frac{ (s-p_{t})^{2}  }{  2 \sigma^{2} } )$$
    - $${\sigma}$$ = $$D/2$$
    - $$s$$ = $$p_{t}$$를 중심으로 하는 window 내의 위치
    - 즉, $$p_{t}$$와 가까울수록 관계성이 높을 것이라고 예상하여, $$p_{t}$$와 가까우면 gaussian distribution값 커져 $$a_{t}$$ 값이 커지고, $$p_{t}$$와 멀면 $$a_{t}$$의 크기를 줄여줬다.

## 4.3 Input-feeding Approach
![input-feeding](https://files.slack.com/files-pri/T1J7SCHU7-F9DP3ECE6/input_feeding.png?pub_secret=d9c200db28)
- global attention과 local attention에서는 각 time step의 alignment $$a_{t}$$가 독립적으로 계산됨
- 하지만 standard MT에서는 어떤 학습과정에서 source word가 target word로 변환되었는지 계속 tracking
- 따라서 attentional NMT model에서도 이전의 alignment를 고려하게 해줌
    - **여태까지 어떤 source word가 관계성이 있다고 판단되었는지도 학습**
- attentional vector $$\widetilde{h_{t}}$$를 다음 input에 같이 넣어줌
    - $$\widetilde{h_{t}}$$ = $$tanh(W_{c}[c_{t};h_{t}])$$
    - 즉, 이전 step에서 attention과 hiddent state를 연산한 것을 input에 같이 넣어줌

-----

# 5. Experiments
- BLEU score로 계산

## 5.1 Training Details
- WMT'14로 학습
- 50k의 frequent vocabulary만 사용, 나머지는 <unk> 처리
- 50자가 넘는 sentence만 학습
- 4-layers LSTM with 1000cells
- 1000 dimensional embedding
- parameter는  [-0.1, 0.1] 범위에서 uniform 분포로 random하게 initialize
- epoch이 지날수록 learning rate를 낮춤
- gradient norm이 5를 초과할 때마다 normalize
- dropout 사용

## 5.2 English-German Results
![result1](https://files.slack.com/files-pri/T1J7SCHU7-F9DP3F96E/result1.png?pub_secret=ae232cbf2a)
- source sentence를 반대로 입력해주었을 때 성능이 더 좋았다.
- dropout을 해줬을 때 성능이 더 좋았다.
- local attention이 global attention보다 성능이 좋았다.
- feed input을 해준 것이 성능이 더 좋았다.
- <unk>을 다른 vector로 바꿨을 때 더 성능이 좋았다.
- ensemble 모델이 성능이 더 좋았다.

## 5.3 German-English Results
![result2](https://files.slack.com/files-pri/T1J7SCHU7-F9DSN4TCM/result3.png?pub_secret=9b4d2839c0)

-----

# 6. Analysis
## 6.1 Learning curves
![learning curve](https://files.slack.com/files-pri/T1J7SCHU7-F9DSN4TCM/result3.png?pub_secret=9b4d2839c0)

## 6.2 Effects of Translating Long Sentences
![long](https://files.slack.com/files-pri/T1J7SCHU7-F9DNGHWBW/result5.png?pub_secret=930e2022c1)
- attention이 추가되면 긴 문장을 더 잘 다룰 수 있다.

## 6.3 Choices of Attentional Architectures
![choice](https://files.slack.com/files-pri/T1J7SCHU7-F9EQNDENB/result6.png?pub_secret=0e82a65f22)

## 6.4 Alignment Quality
![align](https://files.slack.com/files-pri/T1J7SCHU7-F9EQNDENB/result6.png?pub_secret=0e82a65f22)

## 6.5 Sample Translation
![sample](https://files.slack.com/files-pri/T1J7SCHU7-F9EJPE094/result8.png?pub_secret=68efaf2686)

-----

# 7. Conclusion
attention모델이 번역능력에서 우수할 뿐만 아니라과 긴 문장도 다룰 수 있다는 점에서 기존의 모델들보다 뛰어나다.

-----

# 8. Reference
- [http://aclweb.org/anthology/D15-1166](http://aclweb.org/anthology/D15-1166)
- Ybigta deepNLP-study
