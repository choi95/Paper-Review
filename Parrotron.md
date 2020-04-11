# Paper-Review





### Abstract & Intro
입력 spectrum을 다른 spectrum으로 직접 매핑하는 end-to-end 음성 변환 모델이다. 
인코더, 스펙트로그램 및 음소 디코더로 구성 되며, 보코더가 음성으로 변환하는 역할을 맡는다. 
악센트, 프로소디 및 배경 소음에 관계없이 모든 입력 스피커의 음성을 고정 된 억양과 
일관된 조음 및 단일 표준 대상 스피커의 음성으로 정규화하도록 학습하였고, 
또한 청각 스피커(목소리에 장애가 있는)로부터의 음성을 표준화하도록 조정하였으며, 
이는 성공하고, 이를 통한 비정형 목소리를 음성 인식기를 통한 WER measurement및 
무작위 대상으로부터의 Mean Opinion Score (MOS) 측정을 통해 제안한 모델이 명료도 및 
자연스러움이 얼만큼 개선되었는지 보여주고 있다. 


### Model Architecture

타겟 신호를 직접 생성하며 처음부터 합성하는 end-to-end architecture이다
화자는 여러명이고 target 음성의 화자는 1명이라는 것이다. 사용되는 데이터는 쌍을 이루는 입력/출력 음성 발화의 병렬 모음이 필요하다.

전체 모델 아키텍처는 Encoder, Spectrogram decoder, ASR decoder가 사용되고 있다. 

 1) Encoder
최근의 Attention 기반 end-to-end ASR 모델
1: Listen, Attend and Spell
2: Spectral and Prosodic Transformations of Hearing-impaired Mandarin Speech

Tacotron
1: Tacotron2-Natural TTS synthesis by conditioning WaveNet on mel spectrogram predictions,
2: Tacotron-Towards end-to-end speech synthesis 같은 TTS 모델을 기반으로 한다.

Parrotron의 기본 encoder 구성은 
"Sequence-to-sequence models can directly translate foreign speech" 와 유사하다

1-1) Encoder
- 이 논문의 인코더 입력은 80channel log mel spectrogram이다.

- 25ms로 frame length를 두었고, 10ms hop을 사용하여 40%만큼 겹쳤다. 90 symbol로 출력하는데, phoneme기반의 음성인식이다.

- Encoder는 8개의 layer로 이루어져있으며, Time X 80 X 3 tensor형태이다. 

- 그 후, 2개의 CNN + ReLU 활성화를 통과한다. 여기에서 각 CNN은 32kernels로 이루어진 3 X 3 X에서 time X frequency이다. 이 CNN의 stride 는 2 X 2를 사용하였고, 결국 total factor는 4로 downsampling된다.

- 이후 Batch Normalization을 적용한다. CNN~Batch까지 다른 관점에서 바라보면 vgg랑 비슷하다고 볼 수 있지 않을까?

- 그 후, single bidrectional convolution LSTM (CLSTM)을 통과하는데, 이 때 filter 가 1 X 3 으로 통과한다.

- 마지막으로, 각 방향당 256크기의 LSTM을 갖는 BiLSTM을 통과하는데, 이 후 512차원의 Dense를 통과한다. Dense를 통해 Linear Projection을 하고 (512), Batch normalization과 ReLU가 함께 쓰인다.

- 즉 최종 Encoder의 ouptut은 512-dimensonal의 인코더 representation으로 나온다.


1-2) Decoder

64-dimensional의 word embedding된 y_{k-1}가 입력인데, 즉 text가 입력이다. 음성~ -> 인식 이므로 음성이 text로 바뀜. 심볼은 이전 time-step을 이용하며, 512-dimensional의 attention context vector를 사용한다.

- 128개의 Single hidden layer (attention)으로 이루어져있다.

- 이것들이 256-dimensional의 4개의 Unidirectional LSTM으로 통과한다.

- 마지막의 attention context와 LSTM output은 concatenation되어 출력 단어(vocabulary)를 예측하는 softmax로 통과한다.


2) Decoder (Spectrogram decoder)
- 디코더 타겟은 1025-dim STFT이며, 왜 이러냐면... NFFT를 1024으로 사용하고 hop을 25%사용했기 때문에 1025차원의 Spectrum을 얻는다. 마지막 1은 허수부분이다.

- Autoregressive RNN을 사용하여 decoder를 구성하였으며, 1번의 decoder step당 1 frame씩 인코딩 된 입력 sequence에서 출력 spectrogram으로 predict를 진행한다.

- previous(이전의) decoder time step으로부터의 예측은 256 ReLU가 연결된 2개의 layer를 포함하는 Pre-Net을 통과하는 것이 attention이 잘 먹힌다고 밝혔다. ***여기에서 Pre-Net은 이전에 review한 논문에 자세하게 나와 있음***

- Pre-Net 출력과 attention context vector가 연결되어 1024 unit의 2-layers Unidirectional LSTM을 통과함

- LSTM 출력과 attention context vector을 concatenation하여 Linear transformation을 통해 target spectrogram을 예측한다.

- 마지막으로, ***마찬가지로 이전 논문에서의 Post-Net이 나옴*** 5-layers CNN Post-Net을 통과한다. 각각의 Post-Net은 5 x 1 형태의 512 filter를 갖고, BatchNormalization도 있으며, tanh activation 을 통해 출력 된다.

- 예측된 spectrogram에서 오디오 신호를 합성하기 위해 Griffin-Lim 알고리즘을 사용하여 예측된 magnitude와 일치하는 phase를 추정한 뒤 Inverse Short-Time Fourier Transform을 통해 waveform으로 변환한다.

- Griffin-Lim은 음질이 그닥 좋지 않으므로, Griffin-Lim보다 시간이 오래걸리지만 음성의 품질이 더 좋은 WaveRNN을 사용하여 사람들에게 평가하였다.


3) ASR 디코더를 이용한 Multitask Learning

앞서 Translatotron에서의 구조와 비슷하다. 이 ASR decoder의 목적은 기본 언어의 높은 수준의 표현을 동시에 배우기 위해 인코더 네트워크에 multitask learning 기법을 사용하여 공동으로 train을 하면, encoder 의 음성의 feature를 더 잘 배운다고 한다. 쉽게 말해서, Signal-to-Signal로의 변환이 어렵기 때문에 중간에 ASR 디코더를 달아서 음성 부분을 학습시키는 것이다. 인코더 latent 표현에 따라 출력 음성의 (grapheme:자소 or phoneme:음소) transcript를 예측하는 역할을 맡는다. 이러한 multitask learning으로 train된 인코더는 기초적인 transcript에 대한 정보를 유지하는 입력의 latent 표현을 학습하는 것으로 생각할 수 있다.

 

ASR decoder의 input은 이전 step의 grapheme에 대한 64-dimensional 임베딩과(ASR decoder part) 512-dimensional의 attention context를 concatenate하여 생성된다.  그 후 256 unit LSTM layer로 전달 되고, 마지막으로 attention context와 LSTM의 output은 concatenated되어 softmax로 전달되고, 그 후 phoneme으로 출력 될 확률을 예측한다.

 
 



