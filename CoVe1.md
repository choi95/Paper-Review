---
layout: post
title:  "CoVe[1] Learned in Translation: Contextualized Word Vectors(2017) - Review"
date:   2018-02-17 05:24:00 +0900
categories: [deeplearning, rnn, seq2seq, nlp, paperreview]
---

## 1. Abstract
- word vectors를 contextualize 하기 위한 방법을 제시한 논문
    - **contextualize = 문맥화하다**
- Transfer Leaning을 NLP에 적용
    - attentional seq2seq 모델(machine translation)의 deep-LSTM Encoder를 응용
    - Transfer Learning을 통해 얻은 vector가 context vector, 즉, CoVe
- **번역모델의 Encode가 Featrue Extractor의 역할 수행**

-----

## 2. Introduction
- 이미지 분야에서 pretrained 된 layer가 다른 모델에 더해져서 더 좋은 성능을 낼 수 있다.
    - 그 원리를 적용하여 pretrained된 NLP Encoder를 사용하여 성능 개선 시도
- LSTM-based Encoder는 다른 NLP모델에서 많이 사용하므로 이를 활용
- **Transfer-Learning**
    - **다른 모델에서 잘 학습된 Encoder는 다른 모델에 적용되어도 좋은 효과를 발휘할 것이다.**


# 2.1 Architecture
![architecture](https://files.slack.com/files-pri/T1J7SCHU7-F95FQ9EBC/1.png?pub_secret=cda280ac4b)
- 번역모델에서 학습된 Encoder를 다른 NLP모델에 붙여주어 CoVe를 만들고, 원래 vector에 concatenate
- Encoder를 학습시키는 번역 모델의 data set이 많을수록, CoVe를 사용하는 모델의 성능 개선
- MT가 text classification, question answering과는 관련없어 보이지만, 문맥을 이해한다는 점에서 효과적
- 단지 word2vec이나 GloVe를 통해서만 학습된 word vector를 사용하는 것보다 좋은 효과

# 2.2 Machine Translation
번역모델은 어떤 Text를 그대로 다른 형태의 Text로 바꾸는 작업이다. 따라서 Encoding과 Decoding 작업에서 **source language의 정보손실이 거의 없기 때문에 transfer learning에 적합하다.** 또한, MT를 위한 데이터가 많으므로 다른 학습에 비하여 상대적으로 유리하다.

-----

## 3. Related Work
- Transfer Learning
- Neural Machine Translation
- Transfer Learning and Machine Translation
- Transfer Learning in Computer Vision

-----

## 4. Machine Translation Model
- 논문에서는 attentional sequence-to-sequence model for English-to-German translation모델 사용
    - $${W_{1}} = [{a_{1}}, {a_{2}}, ..., {a_{n}}]$$을 $${W_{2}} = [{b_{1}}, {b_{2}}, ..., {b_{m}}]$$으로 바꾸는 과정을 학습

- $$GloVe({W_{1}})$$를 Encoder인 two-layer bidirectional long short-term memory network (MT-LSTM)에 넣어줌.
    - $$h$$ = $$MT-LSTM(GloVe({W_{1}}))$$

- 이전 step들의 결과들을 Decoder 역할을 하는 two-layer unidirectional LSTM에 넣어줌
    - $${h_{t}^{dec}}$$ = $$LSTM([{z_{t-1}};\widetilde{h_{t-1}}], {h_{t-1}^{dec}})$$
    - $$\widetilde{h_{t-1}}$$ = context-adjusted hidden state, $${z_{t-1}}$$ = previous target embedding
  
- attention weights vector $$\alpha$$ 계산
    - $${\alpha_{t}}$$ = $$softmax(H({W_{1}}{h_{t}^{dec}} + {b_{1}}))$$
    - $$H$$ = elements of h stacked along the time dimension

- 최종적으로 Decoder에서는 $$\alpha$$를 이용한 attentional sum과 $${h_{t}^{dec}}$$을 concatenate
    - $$\widetilde{h_{t}}$$ = $$[tanh({W_{2}}{H^{T}}{\alpha_{t}} + {b_{2}});{h_{t}^{dec}}]$$

- output words의 distribution 생성
    - $$p( {b_{t}} \mid X, {b_{1}}, {b_{2}}, \cdots , {b_{t-1}} )$$ = $$softmax( {W_{3}} \widetilde{h_{t}} + {b_{3}} )$$ 

-----

## 5. CoVe
- 위 번역 모델의 Encoder인 MT-LSTM을 가져온 후 학습.
    - $$CoVe(W)$$ = $$MT-LSTM(GloVe(W))$$
- $$\widetilde{W} = [GloVe(W);CoVe(W)]$$의 형태로 concatenate하여 만든 vector를 input으로 사용

-----
  
## 6. Classification with CoVe
![classification](https://files.slack.com/files-pri/T1J7SCHU7-F95EB7L1Z/classification.png?pub_secret=2a34d1153c)
- CoVe의 성능을 시험하기 위하여 위의 classification모델 사용
- pooling의 방식으로는 [max-pooling;mean-pooling;min-pooling;self]의 형태로 만들어줌

-----

## 7. Question Answering with CoVe
- 전체적인 과정은 위와 같지만, ReLU가 아닌 tanh를 activation function으로 사용

-----

## 8. Datasets
![dataset](https://files.slack.com/files-pri/T1J7SCHU7-F95LRMGG5/dataset.png?pub_secret=e96345b6ef)

-----

## 9. Experiments
- character n-gram embeddings 에서 더 효과적이었다.
![e1](https://files.slack.com/files-pri/T1J7SCHU7-F95GSUDHA/e1.png?pub_secret=0b895763a0)

- 같은 모델이라도 어떤 데이터를 사용하는지에 따라서 성능이 다르다.
![e2](https://files.slack.com/files-pri/T1J7SCHU7-F95HB30QJ/e2.png?pub_secret=f1b900de06)
- data set마다 가장 효과적인 모델이 달랐다.

![e3](https://files.slack.com/files-pri/T1J7SCHU7-F96D0D04E/e3.png?pub_secret=f57fd72d63)

-----

## 10. Conclusion
- CoVe는 NLP모델에서 성능을 높이는데 큰 도움이 된다.

-----

## 11. Reference
- [https://arxiv.org/abs/1708.00107](https://arxiv.org/abs/1708.00107)
- Ybigta deepNLP-study
