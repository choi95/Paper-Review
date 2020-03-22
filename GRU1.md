---
layout: post
title:  "GRU[1] Empirical Evaluation of Gated Recurrent Neural Networks on Sequence Modeling(2014) - Review"
date:   2018-02-27 12:15:00 +0900
categories: [deeplearning, rnn, gru, paperreview]
---

## 1. Abstract
GRU와 LSTM의 성능을 비교한 논문이며, 결론적으로 성능이 비슷하다고 한다.

-----

## 2. Introduction
**GRU와 LSTM이 성능은 비슷하지만, GRU가 CPU의 연산량이과 연산에 필요한 parameter 개수에 관해서는 LSTM보다 효율적이다.**

-----

## 3. Background: Recurrent Neural Network
RNN은 vanishing/exploding gradient로 인한 long-term dependency 문제가 있다. 이러한 문제점을 LSTM이나 GRU 같은 gating unit을 통해 극복가능하다.

-----

## 4. Gated Recurrent Neural Networks
![lstm, gru](https://files.slack.com/files-pri/T1J7SCHU7-F9F7AQ152/model.png?pub_secret=11d41cce4e)
- **gate를 통하여 RNN에서 vanishing gradient때문에 멀리 있는 정보가 잘 전달되지 않는 단점 해결**
    - cell state는 통해 큰 연산 없이 계속 network를 통과
    - **즉, 오래된 정보를 잘 전달해준다고 해줄 수 있음**
    - 다른 관점에서 보면 gradient를 잘 전달해주므로 vanishing gradient 문제 해결

# 4.1 [LSTM](https://hwkim94.github.io/deeplearning/rnn/lstm/nlp/paperreview/2018/02/21/LSTM1.html)
![lstm](https://files.slack.com/files-pri/T1J7SCHU7-F9F7APJKW/lstm.png?pub_secret=0ea3a2a4eb)
![Formula](https://files.slack.com/files-pri/T1J7SCHU7-F9BNRBP5J/math2.png?pub_secret=252df47e14)
- $${f_{t}}$$ = forget gate 
    - $$\sigma$$를 통하여 previous state $${c_{t-1}}$$ 얼마나 기억할지 결정
- $${i_{t}}$$ = input gate
    - $$\sigma$$를 통하여 새로운 값 $${g_{t}}$$을 얼마나 반영할지 결정
- $${o_{t}}$$ = output gate
    - $$\sigma$$를 통하여 다음 cell에서 current state $${c_{t}}$$를 얼마나 영향을 줄지 결정
- $${g_{t}}$$ = cell input activation function
    - 새로운 input $${x_{t}}$$과 전달받은 이전의 값$${h_{t-1}}$$을 같이 연산하여 새로운 정보를 생성
- $${c_{t}}$$ = cell state
    - forget gate $${f_{t}}$$와 previous state $${c_{t-1}}$$를 element-wise product하여 $${c_{t-1}}$$가 얼마나 남을지를 결정
    - input gate $${i_{t}}$$와 $${g_{t}}$$를 element-wise product하여 더해줘 current state $${c_{t}}$$ update
- $${h_{t}}$$ = hidden state
    - $${o_{t}}$$와 곱해져서, 다음 cell에 현재의 정보를 넘겨줌
- $$\sigma$$ = sigmoid
    - sigmoid는 0~1 범위의 값을 출력하므로 각 gate에서 얼마나 영향을 줄지를 결정
    - 0을 출력할 경우엔 완전히 영향을 주지않고, 1을 출력할 경우 그대로 정보가 전달됨
- $$\odot$$ = element-wise product

# 4.2 GRU
![gru](https://files.slack.com/files-pri/T1J7SCHU7-F9EMD08KT/total.png?pub_secret=64db233713)
- LSTM의 gate들을 재구성하여 간단해진 architecture

- $$z_{t}$$ = update gate
    - 새로운 정보를 어떻게 구성할지를 결정하는 의미
    - 새로운 input $$x_{t}$$와 previous hidden state를 고려하여 새로운 정보를 얼마나 current hidden state에 반영할지를 결정
    - $$z_{t}$$는 생성된 새로운 정보 $$\widetilde{h_{t}}$$에 곱해져  얼마나 반영될지 결정
    - $$1 - z_{t}$$는 과거의 정보 $$h_{t-1}$$에 곱해져 얼마나 반영될지 결정
    - LSTM의 input gate와 forget gate의 역할을 합친 형태

- $$r_{t}$$ = reset gate
    - 과거의 정보를 t-th time step에서 만들어지는 정보에 얼마나 반영할지만 고려하는 의미, 이전의 기억을 지우는 역할을 하지는 않음
    - 새로운 input $$x_{t}$$과 과거의 정보 $$h_{t-1}$$를 고려했을 때, 과거의 정보 $$h_{t-1}$$를 얼마나 잊을지를 결정
    - 과거의 정보 $$h_{t-1}$$에 곱해져 얼마나 지워지는지 결정
    - LSTM의 forget gate와 비슷해보이지만, 새로운 정보 $$\widetilde{h_{t}}$$ 구성에만 영향을 미친다는 점에서 다른 역할 

- $$\widetilde{h_{t}}$$ = temporal hidden state
    - t-th time step에 형성된 새로운 정보를 의미
    - 과거의 정보 $$h_{t-1}$$를 reset gate $$r_{t}$$에 통과시켜 정보를 지우고, 새로운 input $$x_{t}$$과 연산하여 새로운 정보
 $$\widetilde{h_{t}}$$를 생성

- $${h_{t}}$$ = current hidden state
    - 새로운 정보 $$\widetilde{h_{t}}$$와 이전의 정보 $$h_{t-1}$$를 update gate $$z_{t}$$를 통해 각각 얼마나 반영할지 결정
    - LSTM의 cell state와 hidden state가 합쳐져있는 형태

- $$\sigma$$ = sigmoid
    - sigmoid는 0~1 범위의 값을 출력하므로 각 gate에서 얼마나 영향을 줄지를 결정


# 4.3 Discussion
### 4.3.1 RNN vs LSTM, GRU
- RNN
    - 현재 가지고 있는 content(정보)를 계속 activation을 통해 계속 변형
    - $$\widetilde{h_{t}}$$가 따로 존재하지 않음, 즉, 새로운 input $$x_{t}$$를 그대로 받아들임

- LSTM, GRU
    - **현재 가지고 있는 content(정보)는 건들지 않고, 새로운 content(정보)를 만들어 더해줌. 따라서 과거의 정보가 activation을 통해 바뀌지 않으므로 장기기억을 더 잘한다고 볼 수 있으며, 이전의 content에 직접적으로 연산을 해주지 않으므로 shortcut path를 형성하여 gradient를 잘 전달할 수 있음**
    - 새로운 input $$x_{t}$$과 과거의 정보 $$h_{t-1}$$을 고려하여 새로운 정보 $$\widetilde{h_{t}}$$를 구성함. 이것은 과거의 정보가 영향을 미친다는 철학을 반영

### 4.3.2 LSTM vs GRU
- LSTM
    - output gate를 통하여 현재 가지고 있는 정보를 다음 time step에 얼마나 노출시킬지 조절
    - 이전의 정보를 얼마나 잊을지(forget gate)와 새로운 정보를 얼마나 받아들일지(input gate)가 독립적으로 분리

- GRU
    - 현재 가지고 있는 모든 정보를 다음 time step에 노출시킴
    - **기억의 망각과 갱신이 update gate를 통하여 한 번에 처리됨. 즉, GRU는 과거의 정보를 잊는 것과 새로운 정보를 받아들이는 것은 동시에 발생하는 일이며, 독립적인 작업이 아니라는 철학을 반영**

-----

## 5. Experiments Setting

# 5.1 Tasks and Datasets
- log likelihood를 최대화하는 방향으로 학습
    - $$max \frac{1}{N} \sum_{n=1}^{N} log (p(y \mid x))$$ . 
    - 모든 training set에 대하여 올바를 결과를 도출할 확률을 최대화
- polyphonic music modeling을 위한 dataset 사용
    - Nottingham, JSB Chorales, MuseData and Piano-midi

# 5.2 Models
![result1](https://files.slack.com/files-pri/T1J7SCHU7-F9G3HPHE2/result1.png?pub_secret=6eb810fa1d)

-----

## 6. Results and Analysis
![result2](https://files.slack.com/files-pri/T1J7SCHU7-F9FBGPXFF/result2.png?pub_secret=fa364fca40)
- **성능은 비슷하지만, 연산량과 연산속도 차원에서는 architecture가 더 간단해진 GRU가 조금 더 우수하다**

-----

## 7. Conclusion
일반적인 RNN에 비하여 gated unit은 훨씬 성능이 좋지만, LSTM과 GRU 중에서 어떤 것이 훨씬 뛰어나다고는 말 할 수 없다. 

-----

## 8. Reference
- [https://arxiv.org/abs/1412.3555](https://arxiv.org/abs/1412.3555)
- [http://colah.github.io/posts/2015-08-Understanding-LSTMs/](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)
- [https://brunch.co.kr/@chris-song/9](https://brunch.co.kr/@chris-song/9)
