# Paper-Review

### INTRODUCTION
레이더 데이터를 활용한 시간차에 따른 빠른 기상(강우) 예측이 목적
시공간상(spatiotemproal)의 시퀀스로부터 그다음 시공간상의 시퀀스를 예측
FC-LSTM보다 공간을 잘 고렬할 수 있는 ConvLSTM이라는 새로운 방법을 사용하여 풀었음

### FC-LSTM(FULLY CONNECTED LSTM)
FC-LSTM은 input과 ouput 그리고 state가 모두 1D vector이며 LSTM의 다변량 버전이라고 할 수 있음.


CNN+LSTM이 아니라 LSTM 안에 CNN을 넣어버린다.
그래서  input, output, Ct, Ht 등등이 모두 3D tensor가 됨.
합성곱(*)이 들어간 것이 특징임
그렇기 때문에 기존에 FC-LSTM에서 weight 수가 많이 줄어드는 효과가 생김


### EXPERIMENTS
ConvLSTM이 FC-LSTM보다 시공간상의 관계를더 잘 다루기 때문에 좋게 나옴
Deeper model의 경우에는 더 적은 파라미터로 더 좋은 성능을 내었음(FC-LSTM 대비)
ConvLSTM(9×9)시리즈 모델 두개는 커널 사이를 1×1로 줄여 본 것
FC-LSTM 대비해서 성능이 더 좋으면서도 파라미터는 적은 장점이 크게 나타남.
(10배가 적으면서 성능은 20%더 좋음)


