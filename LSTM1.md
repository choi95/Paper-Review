---
layout: post
title:  "LSTM[1] Long Short Term Memory Recurrent Neural Network Architectures for Large Scale Acoustic Modeling(2014) - Review"
date:   2018-02-21 00:15:00 +0900
categories: [deeplearning, rnn, lstm, nlp, paperreview]
---

원래는 LSTM이란 개념을 처음 제시한 논문을 읽고 요약하려했지만, 다른 논문과 자료들에 더 정리가 잘 되어있는 것 같아서 이 논문을 읽기로 했다. (그 논문이 길어서 그런거 절대 아님...) 혹시 1997년의 논문을 읽고싶으시면 [Long Short-Term Memory](http://www.bioinf.jku.at/publications/older/2604.pdf) 참조

## 1. Abstract
- LSTM은 RNN의 specific architecture
    - time sequential data를 처리하고 long-range dependecy를 통하여 기존 RNN보다 성능을 개선
- 이 논문에서는 ASGD(asynchronous gradient descent)를 통해 LSTM의 분산학습방법 소개
- 이 논문에서는 acoustic speech recognition data에 대해 모델링

-----

## 2. Introduction
- Speech는 각각의 timescale마다 다른 상관관계가 있는 복잡한 time-varying signal
- DNN(Deep Neural Net)은 정형데이터와 window 안에 있는 데이터만 처리 가능
    - speech가 보유한 speaking rates, length 등 다양성을 반영하지 못한다.
- RNN은 이전 time step을 input으로 받기 때문에 이전의 정보를 활용 가능
- LSTM은 RNN의 단점을 보완한 architecture
- BLSTM은 input sequence를 양방향으로 넣어주는 architecture로, LSTM보다 좋은 성능

-----

## 3. LSTM Network Architectures
![LSTM architectures](https://files.slack.com/files-pri/T1J7SCHU7-F9BS5S5GV/lstm2.png?pub_secret=d71e6c41c0)

# 3.1 Conventional LSTM
![LSTM](https://files.slack.com/files-pri/T1J7SCHU7-F9BSW5T3P/lstm.png?pub_secret=f49c1e8a0b)
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

- **gate를 통하여 RNN에서 vanishing gradient때문에 멀리 있는 정보가 잘 전달되지 않는 단점 해결**
    - cell state는 통해 큰 연산 없이 계속 network를 통과
    - **즉, 오래된 정보를 잘 전달해준다고 해줄 수 있음**
    - 다른 관점에서 보면 gradient를 잘 전달해주므로 vanishing gradient 문제 해결

# 3.2 Deep LSTM
- LSTM layer를 여러개 쌓아서 구성
- input의 time scale 차이를 학습할 수 있다.
- 하나의 cell에서 parameter를 늘리는 것보다 LSTM cell 여러 개를 stack하는 것이 효율적

# 3.3 LSTMP(Long Short-Term Memory Projected) - LSTM with Recurrent Projection Layer
![LSTMP](https://files.slack.com/files-pri/T1J7SCHU7-F9CJ6TMPG/lstm1.png?pub_secret=12f6cf44ff)
- LSTM layer 이후에 linear projection layer
    - $${h_{t}}$$에 $$W$$를 곱해 projection, 그리고 이 값을 output으로 사용
- previous hidden state뿐만 아니라, output이 무엇으로 나왔는지도 고려

# 3.4 Deep LSTMP
- nonlinear가 많아지기 때문에 overfitting 방지 가능
- cell이 많아지기 때문에 memory size가 커짐

-----

## 4. Distributed Training: Scaling up to Large Models with Parallelization
- multicore CPU사용
- ASGD(asynchronous gradient descent)를 통해 LSTM의 분산 학습
- LSTM의 parameter를 담고 있는 shared, distributed parameter server를 통해 worker가 통신
- 각 worker가 gradient를 계산하면 partitioned되어 server로 보내짐

-----

## 5. Experiments
![result1](https://files.slack.com/files-pri/T1J7SCHU7-F9BQQ9BEG/r1.png?pub_secret=c640b5d6a5)
![result2](https://files.slack.com/files-pri/T1J7SCHU7-F9BQQA4PN/r2.png?pub_secret=1b6b1e63dd)
- LSTMP 모델이 LSTM모델보다 성능이 좋다.
- LSTMP 모델도 깊어질수록 성능이 좋다.

-----

## 6. Conclusion
- Deep LSTMP이 RNN이나 LSTM보다 parameter를 더 잘 학습하므로 성능이 좋다. 
- ASGD를 사용할 경우, 병렬처리가 가능하므로 학습속도가 훨씬 빨라진다.  

-----

## 7. Reference
- [http://www.isca-speech.org/archive/archive_papers/interspeech_2014/i14_0338.pdf](http://www.isca-speech.org/archive/archive_papers/interspeech_2014/i14_0338.pdf)
- [https://ratsgo.github.io/natural%20language%20processing/2017/03/09/rnnlstm/](https://ratsgo.github.io/natural%20language%20processing/2017/03/09/rnnlstm/)
- [https://brunch.co.kr/@chris-song/9](https://brunch.co.kr/@chris-song/9)
- [http://www.whydsp.org/280](http://www.whydsp.org/280)
- [http://solarisailab.com/archives/1667](http://solarisailab.com/archives/1667)

