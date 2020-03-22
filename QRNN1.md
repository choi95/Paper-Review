---
layout: post
title:  "QRNN[1] Quasi Recurrent Neural Networks(2016) - Review"
date:   2018-03-02 15:24:00 +0900
categories: [deeplearning, rnn, qrnn, paperreview]
---

## 1. Abstract
**RNN은 sequential data를 처리하는 것에서 강력하지만, 각 time step에서 previous hidden state를 사용하여 current hidden state를 연산해야하므로 병렬처리가 되지않기 때문에 데이터를 처리하는 것에 시간이 오래걸린다는 단점이 있다. QRNN은 convolution과 pooling을 통해 sequential data를 병렬처리하며 처리시간을 단축했다.**

-----

## 2. Introduction
QRNN은 CNN과 RNN의 장점을 섞어놓은 Architecture이다. CNN처럼 time step과 mini-batch를 통한 병렬처리가 가능하고, RNN처럼 각 element의 순서에 의존한다.

-----

## 3. Model
# 3.1 Architecture
![model](https://files.slack.com/files-pri/T1J7SCHU7-F9GPA11MX/model.png?pub_secret=b489acf16b)
- QRNN의 layer는 convolution, pooling layer의 subcomponent로 구성
    - convolution layer에서는 masked convolution을 사용하여 previous input을 반영하여 current information 형성
    - pooling layer에서는 hidden state를 만들어 previous hidden state가 current hidden state에 영향을 미치도록 함
    - 이때, masked convolution을 사용하였으므로 previous information만 current information에 영향을 미치게 됨 

- input
    - $$X \in R^{T \times n}$$ = $$(x_{1}, x_{2}, \cdots , x_{T})$$
    - $$X \in R^{T \times n}$$ = $$(x_{1}, x_{2}, \cdots , x_{T})$$ 
    - $$n$$-dimensional vector $$T$$개로 구성된 sequence

- masked convolution
    - $$z_{t}$$ = $$tanh(W_{z}^{1}x_{t-k+1} + W_{z}^{2}x_{t-k+2} + \cdots + W_{z}^{k}x_{t})$$
    - $$f_{t}$$ = $$\sigma (W_{f}^{1}x_{t-k+1} + W_{f}^{2}x_{t-k+2} + \cdots + W_{f}^{k}x_{t})$$
    - $$o_{t}$$ = $$\sigma (W_{o}^{1}x_{t-k+1} + W_{o}^{2}x_{t-k+2} + \cdots + W_{o}^{k}x_{t})$$
    - $$Z \in R^{T \times m}$$ = $$tanh(W_{z} \ast X)$$ 
    - $$F$$ = $$\sigma (W_{f} \ast X)$$ 
    - $$O$$ = $$\sigma (W_{o} \ast X)$$ 
    - $$\ast$$ = masked pooling
    - $$W_{z}, W_{f}, W_{o} \in R^{k \times n \times m}$$ .
    - $$m$$개의 $$k$$-width filter
    - 미래의 token을 예측해야하므로 미래의 정보에 의존해서는 안됨 
    - 따라서, t-th time step에서는 t-th element이전만 볼 수 있게 해야함
    - 만약 filter width = $$k$$라면, $$x_{t-k+1}, \cdots , x_{t}$$까지만 convolution하는 것

- f-pooling 
    - $$h_{t}$$ = $$f_{t} \odot h_{t-1} + (1- f_{t}) \odot z_{t}$$

- fo-pooling
    - $$c_{t}$$ = $$f_{t} \odot c_{t-1} + (1- f_{t}) \odot z_{t}$$
    - $$h_{t}$$ = $$o_{t} \odot c_{t}$$

- ifo-pooling
    - $$c_{t}$$ = $$f_{t} \odot c_{t-1} + i_{t} \odot z_{t}$$
    - $$h_{t}$$ = $$o_{t} \odot c_{t}$$


# 3.2 Variants
### 3.2.1 Regularization
- zone out 기법 사용
    - RNN구조의 Regularization 방법
    - dropout은 Bernoulli확률로 connection을 끊어 network의 일부만 학습시키는 방식이지만, zone out에서는 Bernoulli확률로 전체적인 network를 학습을 시키지 않음

### 3.2.2 Densely-Connected Layer
- DenseNet 기법 사용하여 QRNN layer를 쌓았음
    - DenseNet이란, skip connection을 사용한다는 점에서 resNet과 비슷하지만 resNet은 바로 전의 layer를 'add' 해준다. 하지만, DenseNet에서는 이전까지의 모든 layer를 concat해주는 방식으로 정보를 보존한다.

### 3.3.3 Encoder-Decoder Model 
![model2](https://files.slack.com/files-pri/T1J7SCHU7-F9H9F3QFN/model2.png?pub_secret=aa348b6c08)
- Encoder
    - 기존 QRNN과 같으며, 각 layer마다 last hidden state $$h_{T}^{l}$$를 Decoder의 각 pooling layer로 넣어줌

- Decoder
    - Encoder의 각 layer마다 생성된 last hidden state $$h_{T}^{l}$$를 각 pooling layer의 time-step마다 넣어줌
    - $$Z^{l}$$ = $$tanh(W_{z}^{l} \ast X^{l} + V_{z}^{l} \widetilde{h_{T}^{l}})$$ 
    - $$F^{l}$$ = $$\sigma (W_{f}^{l} \ast X^{l} + V_{f}^{l} \widetilde{h_{T}^{l}})$$ 
    - $$O^{l}$$ = $$\sigma (W_{o}^{l} \ast X^{l} + V_{0}^{l} \widetilde{h_{T}^{l}})$$ 
    - $$l$$ = $$l$$-th layer, $$h_{T}^{l}$$ = Encoder의 $$l$$-th layer의 마지막 hidden state

- Attention
    - $$a_{st}$$ = $$softmax(c_{t}^{L} \cdot \widetilde{h_{s}^{L}})$$
    - $$k_{t}$$ = $$\sum a_{st} \widetilde{h_{s}^{L}}$$
    - $$h_{t}^{L}$$ = $$o_{t} \odot (W_{k} k_{t} + W_{c} c_{t}^{L})$$
    - Encoder와 Decoder의 마지막 layer인 $$L$$-th layer의 각 time step의 hidden state를 dot product한 후 softmax
    - softmax 후에 각 hidden state를 가중합을 구하므로 t-th target과 관련이 큰 input들의 hidden state만 영향력을 가짐
    - 최종적으로, Decoder의 마지막 layer $$L$$의 t-th hidden state $$h_{t}^{L}$$의 계산에 t-th target과 관련 있는 input들이 영향을 주게 됨
    - 즉, Encoder의 s-th input과 Decoder의 t-th target이 얼마나 관련있나를 학습하게 됨  


-----

## 4. Experiment
# 4.1 Sentiment Classification
![result1](https://files.slack.com/files-pri/T1J7SCHU7-F9H854HS7/result1.png?pub_secret=5087573514)
- 성능은 비슷했지만, 속도가 3배 이상 빨랐음

# 4.2 Language Modeling
![result2](https://files.slack.com/files-pri/T1J7SCHU7-F9H854XA7/result2.png?pub_secret=a40f8db85e)
- 더 적은 parameter로 비슷한 성능을 낼 수 있다.
- zoneout이 있으면 overfitting을 방지해준다.

# 4.3 Character-level Neural Machine Translation
![result3](https://files.slack.com/files-pri/T1J7SCHU7-F9JCLD6T1/result3.png?pub_secret=7a731263cb)
- 같은 조건의 LSTM보다 성능이 좋다.

-----

## 5. Related Work
- T-RNN, T-LSTM, T-GRU
- ByteNet
- CNN

-----

## 6. Conclusion
QRNN은 문맥 이해와 병렬 처리 2가지 장점을 모두 가졌으며, LSTM과 견줄정도로 성능이 좋다.

-----

## 7. Reference
- [https://arxiv.org/abs/1611.01576](https://arxiv.org/abs/1611.01576)
- [https://jayhey.github.io/deep%20learning/2017/10/13/DenseNet_1/](https://jayhey.github.io/deep%20learning/2017/10/13/DenseNet_1/)
- Ybigta deepNLP-study
