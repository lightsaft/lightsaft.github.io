---
marp: true
---

# Audio Source Separation: an overview

### Woosung Choi  @ [Intelligence Engineering Lab, Korea University](http://intelligence.korea.ac.kr/)


- Code and models are available [online](https://github.com/ws-choi/Conditioned-Source-Separation-LaSAFT).

- You can also check separated samples from [here](https://lasaft.github.io).
- [opening bgm](/slide/gaudio/music/)
- slide url: https://lightsaft.github.io/slide/biglab/

---

## Contents

### 1. Task Definition: *(Conditioned)* Source Separation
### 2. Part 1: Frequency Transformation Blocks (FTBs)
  - **review**: U-Net for Spectrogram-based Singing Voice Separation
  - **motivation**: Spectrogram $\neq$ Image
  - **solution:** Frequency Transformation Blocks
   
### 3. Part 2: LaSAFT for Conditioned Source Separation
  - **review**: Conditioned-U-Net (C-U-Net) for Conditioned Source Separation
  - **motivation**: Extending FTB to Conditioned Source Separation
  - **solution:** Latent Instrumant Attentive Frequency Transformation  Block (LaSAFT)
  - **how to modulate latent features**: more complex manipulation method than FiLM

---

## 1. Task Definition: Source Separation

- Separates signals of the specific source from a given mixed-signal
  - Music Source Separation, Speech Enhancement...
 
- Categories of Source separation models

  - A *Dedicated model*: is dedicated to a single instrument.
  - A *Multi-head model*: generates several outputs at once with a multi-head
  - A *Conditioned model*: separates different instruments with the aid of the control mechanism
    - Input: an input audio track $A$ and a one-hot encoding vector $C$ that specifies which instrument we want to separate
    - Output: separated track of the target instrument

---

## 1. Task Definition

- Digital Audio Signal Processing (sr:44100Hz)
  - ![](https://upload.wikimedia.org/wikipedia/commons/thumb/0/04/Digital.signal.discret.svg/330px-Digital.signal.discret.svg.png)

  - Linear Audio Mixing System
    - $M[t]=\sum S^{(i)}[t] + N[t]$
  
  - Audio Source Separation 
    - $ASS(M[t])=\{S^{(0)}, S^{(1)}, ..., S^{(n)}\}$


---


## 1. Task Definition: Stretegies

- 그렇다면 음원분리를 어떻게 접근해야 할까?

  - 고음역대를 filtering 해줌으로써 저음역대의 악기 (kick + bass) 만 남기는 [예제](https://youtu.be/a6m35sGt230?t=145)

- 이러한 원리를 조금 더 확장시켜보면, 
  - 내가 원하는 악기가 있을만한 주파수대만 남기고 다른 주파수는 filtering 함으로써 음원분리를 수행할 수 있지 않을까하는 생각이 든다.

- 말은 쉽다. 

---

## 1. Task Definition: Challenges


- 다만 약간의 걸림돌이 있다면

  - kick과 bass의 경우 다른 악기와 잘 겹치지 않는 저음역대라 분리가 쉬움
  - 보컬, 피아노, 그리고 기타 같은 세 악기는 음역대가 많이 겹치는 편
    - **시간대에 상관없는** static한 주파수 filtering은 곤란

- 이를 통해 얻을 수 있는 결론:

  - 주파수 기반의 음원 분리를 '잘' 하려거든, 
    - 노래를 들어보고,
    - 시간대 별로 내가 남겨야 할 주파수를 타게팅한뒤
    - 나머지 주파수를 손봐야한다.

---

## 1. Task Definition: DJ Drop the beat!

- 음향 전문가의 수작업
  - Audio Equalizer - Eliminate signals with unwanted frequencies
  ![](https://imgur.com/u0AoskM.png)
- 주파수 기반의 음원 분리를 '잘' 하려거든, 
  - 노래를 들어보고,
  - 시간대 별로 내가 남겨야 할 주파수를 타게팅한뒤
  - 나머지 주파수를 손봐야한다.

- 자동화...? => Machine Learning...!

---

## 1. Task Definition: Challenges (2)

- 또 한가지 걸림돌: 컴퓨터에게는 소리의 높낮음이 없다

  - 사람은 달팽이관 덕분에 소리의 높낮음이 감지됨
    - 높다, 낮다, 섞였다 등을 감지
    -  이 지점에서 피아노와 보컬이 같이 나오는구나 쯤은 알 수 있음
  - 컴퓨터에게 소리는 시간의 흐름에 따라 값이 달라지는 수열일 뿐
    - 소리의 높고 낮음 따위는 인지하지 못한다.
    - 즉, 컴퓨터에게는 주파수 정보가 없다.

- 이럴 때 사용하는 방법이 [Spectrogram Analysis](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%8E%99%ED%8A%B8%EB%A1%9C%EA%B7%B8%EB%9E%A8)이다.

---

## 1. Task Definition: Spectrogram

- Spectrogram을 완전히 이해하기 위해서는 
  - Fourier Transform을([tutorial 1](https://github.com/Intelligence-Engineering-LAB-KU/Seminar/blob/master/summer_2020/0721_wschoi_Fourier_anlysis_Part1.ipynb), [tutorial 2](https://github.com/Intelligence-Engineering-LAB-KU/Seminar/blob/master/summer_2020/0721_wschoi_Fourier_analysis_Part2.ipynb))에 대해 먼저 알아야한다.

- 그러나 이를 설명하는 것은 배보다 배꼽이 크므로 무슨 역할을 하는지만 알아보자.

  - TLDR: Fourier Analysis 
    - 시간에 따라 변화하는 값인 신호정보를 시간-주파수 형식으로 바꿔줌

    ![width:300](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Spectrogram_of_violin.png/642px-Spectrogram_of_violin.png)

---

## 1. Task Definition: Spec2Spec!
- Spec2Spec: Mixture => Target Spectrogram
  ![height:500](https://imgur.com/ZHXdV38.png)

---

## 2. Part 1: Frequency Transformation Blocks (FTBs)
1. **review**: a (*dedicated*) U-Net for Spectrogram-based Singing Voice Separation
2. **motivation**: Spectrogram $\neq$ Image
     - What's wrong with CNNs and spectrograms for audio processing?
     - Alternatives: 1-D CNNs, Dilated CNNs, FTBs, ...
3. **solution:** Frequency Transformation Blocks
     - Employing Fully-Connected (FC) Layers to capture Freq-to-Freq Dependencies
     - (empirical results) Injecting FCs, called FTBs,  into a Fully 2-D Conv U-Net significantly improves SDR performance
   
---

## 2.1. Review: U-Net for Spectrogram-based Source Separation

- U-Net: an encoder-decoder structure with symmetric skip connections
  - These symmetric skip connections allow models to recover fine-grained details of the target object during decoding effectively.
  ![width:500](https://imgur.com/ua3qkU4.png)

- Originally proposed for ***Medical Image Segmentation***
  - can also be viewed as an Image-to-Image Translation
  - The original U-Net is fully *2-d convolutional*



---

## 2.1. Review: U-Net for Spectrogram-based Source Separation

- Audio Equalizer - Eliminate signals with unwanted frequencies
![](https://imgur.com/u0AoskM.png)

- Spectrogram-based Source Separation
  1. Apply Short-Time Fourier Transform (STFT) on a mixture waveform to obtain the input spectrograms.
  2. Estimate the vocal spectrograms based on these inputs 
  3. Restore the vocal waveform with inverse STFT (iSTFT).


---

## 2.1. Review: U-Net for Spectrogram-based Source Separation

- Spec2Spec (Masking-based or Direct Estimation)
  ![height:500](https://imgur.com/ZHXdV38.png)


---


## 2.1. Review: U-Net For Spectrogram-based Source Separation

- Naive Assumption
  - Assuimg a spectrogram is a two (left and right) - channeled image
  - Spectrogram-based Source Separation can be viewed as an Image-to-Image Translation
  ![width:500](https://camo.githubusercontent.com/e2ca5fce45aafa29442a625b94f8987fbfeba61e624daeb3febc24a678772ea9/68747470733a2f2f706963342e7a68696d672e636f6d2f76322d38646638636164316466343765346134626537363533373831353636333335325f31323030783530302e6a7067)

---

## 2.1. Review: U-Net For Spectrogram-based Source Separation (2)

- ..., and it works...!
  - Jansson, A., et al. "Singing voice separation with deep U-Net convolutional networks." 18th International Society for Music Information Retrieval Conference. 2017.
  - Takahashi, Naoya, and Yuki Mitsufuji. "Multi-scale multi-band densenets for audio source separation." 2017 IEEE Workshop on Applications of Signal Processing to Audio and Acoustics (WASPAA). IEEE, 2017.

- Recall the assumption of this approach:
  - Assuming a spectrogram is a two (left and right) - channeled image
  - Spectrogram-based Source Separation $\approx$ Image-to-Image Translation
  - (empirical results) Fully 2-D Convs can provide promising results

---

## 2.2. Spectrogram $\neq$ Image

- [What's wrong with CNNs and spectrograms for audio processing?](https://towardsdatascience.com/whats-wrong-with-spectrograms-and-cnns-for-audio-processing-311377d7ccd)
  - The axes of spectrograms do not carry the same meaning
    - spatial invariance that 2D CNNs provide might not perform as well
  - The spectral properties of sounds are non-local
    - Periodic sounds are typically comprised of a fundamental frequency and a number of **harmonics** which are spaced apart by relationships dictated by the source of the sound. It is the mixture of these harmonics that determines the timbre of the sound.
  ![width:300](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Spectrogram_of_violin.png/642px-Spectrogram_of_violin.png)

---

## 2.2. What's wrong with CNNs and spectrograms for audio processing?

- Yin, Dacheng, et al."**PHASEN**: A Phase-and-Harmonics-Aware Speech Enhancement Network." AAAI. 2020.
  > Non-local correlations exist in a T-F spectrogram along the
frequency axis. A typical example is the correlations among harmonics ... However, simply stacking several 2D convolution layers with small kernels cannot capture such global correlation. 
- Park, Soochul, and Ben Sangbae Chon. "GSEP: A robust vocal and accompaniment separation system using gated CBHG module and loudness normalization." arXiv preprint arXiv:2010.12139 (2020).
  > The two-dimensional convolution network used in the Spleeter for a frequency component misses the useful information at lower or higher frequency components which is out of the kernel range.
  
---

## 2.2. Source Separation

- Harmonics
  - ![](https://imgur.com/Bsdv5VI.png) ![](https://imgur.com/wM3k1iL.gif)

- Timbre of 'Singing Voice' - decided by resonance patterns

  ![](https://imgur.com/zG32WnQ.png)


---

## 2.2. Alternatives

- 1-D CNNs
  - Liu, Jen-Yu, and Yi-Hsuan Yang. "Dilated convolution with dilated GRU for music source separation." arXiv preprint arXiv:1906.01203 (2019).
- Dilated Convolutions
  - Takahashi, Naoya, and Yuki Mitsufuji. "D3Net: Densely connected multidilated DenseNet for music source separation." arXiv preprint arXiv:2010.01733 (2020).
- ***FTBs***: Frequency Transformation Blocks
  - **PHASEN**, ours
- RNNs: $\sim$ FTBs
  - Takahashi, Naoya, Nabarun Goswami, and Yuki Mitsufuji. "Mmdenselstm: An efficient combination of convolutional and recurrent neural networks for audio source separation." 2018 16th International Workshop on Acoustic Signal Enhancement (IWAENC). IEEE, 2018.

---

## 2.2. Receptive Field: 2-D Conv Layer vs. FC Layer

- [2-D Conv](https://medium.com/mlreview/a-guide-to-receptive-field-arithmetic-for-convolutional-neural-networks-e0f514068807)

![](https://imgur.com/CdgW7Rx.png)

- A single Fully-Connected Layer 
  - can capture every freq-to-freq correlation!

---


## 2.3. Our Approach: Injecting FTBs into U-Nets

- ***FTBs***: Frequency Transformation Blocks

  - An FTB called Time-Distributed Fully-connected Layer ([TDF](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS/blob/master/paper_with_code/Paper%20with%20Code%20-%203.%20INTERMEDIATE%20BLOCKS.ipynb)): 
    ![width:800](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS/raw/cc6fb8048da2af3748aece6ae639af51b93c0a84/paper_with_code/img/tdf.png)

  - Choi, Woosung, et al. "Investigating u-nets with various intermediate blocks for spectrogram-based singing voice separation." 21th International Society for Music Information Retrieval Conference, ISMIR, Ed. 2020.

---

## 2.3. Time-Distributed Fully-connected Layer

```python 
import torch
import torch.nn as nn

class TDF(nn.Module):
    ''' [B, in_channels, T, F] => [B, in_channels, T, F] '''
    def __init__(self, channels, f, bf=16, bias=False, min_bn_units=16):
        
        '''
        channels: # channels
        f: num of frequency bins
        bf: bottleneck factor. if None: single layer. else: MLP that maps f => f//bf => f 
        bias: bias setting of linear layers
        '''
        
        super(TDF, self).__init__()

          bn_unis = max(f//bf, min_bn_units)
          self.tdf = nn.Sequential(
              nn.Linear(f, bn_unis, bias),
              nn.BatchNorm2d(channels),
              nn.ReLU(),
              nn.Linear(bn_unis, f, bias),
              nn.BatchNorm2d(channels),
              nn.ReLU()
          )
            
    def forward(self, x):
        return self.tdf(x)

```

---

## 2.3. Injecting TDFs into a U-Net framework

- Building Block TFC-TDF: Densely connected 2-d Conv (TFC) with TDFs

  ![width:700](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS/raw/cc6fb8048da2af3748aece6ae639af51b93c0a84/paper_with_code/img/tfctdf.png)

- U-Net with TFC-TDFs

  ![width:300](https://imgur.com/tmhMpqL.png)   + ![width:500](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS/raw/cc6fb8048da2af3748aece6ae639af51b93c0a84/paper_with_code/img/tfctdf.png)

---

## 2.3. Results?

- Ablation (n_fft = 2048)
  - U-Net with 17 TFC blocks: SDR 6.89dB
  - U-Net with 17 TFC-**TDF** blocks: SDR 7.12dB (+0.23 dB)

- Large Model (n_fft = 4096)
![](https://imgur.com/Dby4Rkw.png)

---

## 2.3. Why does it work?: Weight visualization

- freq patterns of different sources captured by TDFs, of FTBs

![width:500](https://imgur.com/bH4cgKq.png) ![width:500](https://imgur.com/8fSULqe.png)

---

## 2.3. ISMIR 2020

![width:600](https://imgur.com/C5XUq5W.png)

---

## 3. Part 2: LaSAFT for Conditioned Source Separation

  - **review**: Conditioned-U-Net (C-U-Net) for Conditioned Source Separation
  - **motivation**: Extending FTB to Conditioned Source Separation
    - Naive Extention: Injecting FTBs into C-U-Net?
    - (emprical results) It works, but ...
  - **solution:** Latent Instrumant Attentive Frequency Transformation  Block (LaSAFT)
  - **how to modulate latent features**: more complex manipulation method than FiLM

---

## 3.1. Conditioned Source Separation

- Task Definition
  - Input: an input audio track $A$ and a one-hot encoding vector $C$ that specifies which instrument we want to separate
  - Output: separated track of the target instrument
- Method: Conditioning Learning
  - can separate different instruments with the aid of the **control mechanism**. 
  - Conditioned-U-Net (C-U-Net)
    - Meseguer-Brocal, Gabriel, and Geoffroy Peeters. "CONDITIONED-U-NET: INTRODUCING A CONTROL MECHANISM IN THE U-NET FOR MULTIPLE SOURCE SEPARATIONS." Proceedings of the 20th International Society for Music Information Retrieval Conference. 2019.

---

## 3.1. C-U-Net

- Conditioned-U-Net extends the U-Net by exploiting Feature-wise Linear Modulation (FiLM)

  ![width:700](https://github.com/gabolsgabs/cunet/raw/master/.markdown_images/overview.png)


---

## 3.1. C-U-Net: Feature-wise Linear Modulation

  ![width:1200](https://github.com/gabolsgabs/cunet/raw/master/.markdown_images/c-u-net.png)

---

## 3.2. Naive Extention: Injecting FTBs into C-U-Net?

- Baseline C-U-Net + TFC-TDFs

  ![width:600](https://imgur.com/NpB5BpU.png)

- TFC vs TFC-TDF
  ![width:600](https://imgur.com/e6giOhG.png)

---

## 3.2. Naive Extention: Above our expectation

- TFC vs TFC-TDF
  ![width:600](https://imgur.com/e6giOhG.png)

- Although it does improve SDR performance by capturing common frequency patterns observed across all instruments,
  - Merely injecting an FTB to a CUNet **does not inherit the spirit of FTBs**

- We propose the Latent Source-Attentive Frequency Transformation (LaSAFT), a novel frequency transformation block that can capture instrument-dependent frequency patterns by exploiting the scaled dot-product attention

---

## 3.3. LaSAFT: Motivation

- Extending TDF to the Multi-Source Task
  - Naive Extension: MUX-like approach
    - A TDF for each instrument: $\mathcal{I}$ instrument => $\mathcal{I}$ TDFs
    ![](https://upload.wikimedia.org/wikipedia/commons/e/e0/Telephony_multiplexer_system.gif)

  - However, there are much more 'instruments' we have to consider in fact
    - female-classic-soprano, male-jazz-baritone ... $\in$ 'vocals' 
    - kick, snare, rimshot, hat(closed), tom-tom ... $\in$ 'drums'
    - contrabass, electronic, walking bass piano (boogie woogie) ... $\in$ 'bass'  

---

## 3.3. Latent Source-attentive Frequency Transformation

- We assume that there are  $\mathcal{I}_L$ latent instruemtns
  - string-finger-low_freq
  - string-bow-low_freq
  - brass-high-solo
  - percussive-high
  - ...
- We assume each instrument can be represented as a weighted average of them
  - bass: 0.7 string-finger-low_freq + 0.2 string-bow-low_freq + 0.1 percussive-low

- LaSAFT
  - $\mathcal{I}_L$ TDFs for  $\mathcal{I}_L$ latent instruemtns
  - attention-based weighted average


---

## 3.3. LaSAFT: Extending TDF to the Multi-Source Task (1)

![width:600](https://imgur.com/vQNgttJ.png)

- duplicate $\mathcal{I}_L$ copies of the second layer of the TDF,  where $\mathcal{I}_L$ refers to the number of ***latent instruments***.
  - $\mathcal{I}_L$ is not necessarily the same as $\mathcal{I}$ for the sake of flexibility
- For the given frame $V\in \mathbb{R}^F$, we obtain the $\mathcal{I}_L$ latent instrument-dependent frequency-to-frequency correlations, denoted by $V'\in \mathbb{R}^{F \times \mathcal{I}_L}$.

---

## 3.3. LaSAFT: Extending TDF to the Multi-Source Task (2)

![width:600](https://imgur.com/vQNgttJ.png)

- The left side determines how much each ***latent source*** should be attended
- The LaSAFT takes as input  the instrument embedding $z_e \in \mathbb{R}^{1 \times E}$. 
- It has a learnable weight matrix $K\in \mathbb{R}^{ \mathcal{I}_L \times d_{k}}$, where we denote the dimension of each instrument's hidden representation by $d_{k}$.
- By applying a linear layer of size $d_{k}$ to $z_e$, we obtain $Q \in \mathbb{R}^{d_{k}}$.

---

## 3.3. LaSAFT: Extending TDF to the Multi-Source Task (3)

![width:600](https://imgur.com/vQNgttJ.png)

- We now can compute the output of the LaSAFT as follows:

  - $Attention(Q,K,V') = softmax(\frac{QK^{T}}{\sqrt{d_{k}}})V'$

- We apply a LaSAFT after each TFC in the encoder and after each Film/GPoCM layer in the decoder. We employ a skip connection for LaSAFT and TDF, as in TFC-TDF.

---

## 3.3. Effects of employing LaSAFTs instead of TFC-TDFs

![width:600](https://imgur.com/sPVDDzZ.png)

---

## 3.4. GPoCM: more complex manipulation method than FiLM

- FiLM (left) vs PoCM (right)

![width:550](https://imgur.com/A3kAxVS.png) ![width:550](https://imgur.com/9A4otVA.png)

- PoCM is an extension of FiLM. 
  - while FiLM does not have inter-channel operations
  - PoCM has inter-channel operations

---

## 3.4. GPoCM: more complex manipulation method than FiLM (2)

- PoCM is an extension of FiLM

  - $FiLM(X^{i}_{c}|\gamma_{c}^{i},\beta_{c}^{i}) =  \gamma_{c}^{i} \cdot X^{i}_{c} + \beta_{c}^{i}$

  - $PoCM(X^{i}_{c}|\omega_{c}^{i},\beta_{c}^{i}) = \beta_{c}^{i} + \sum_{j}{\omega_{cj}^{i} \cdot X^{i}_{j}}$
    - where $\gamma_{c}^{i}$ and $\beta_{c}^{i}$ are parameters generated by the condition generator, and $X^{i}$ is the output of the $i^{th}$ decoder's intermediate block, whose subscript refers to the $c^{th}$ channel of $X$

    ![width:500](https://imgur.com/9A4otVA.png)

---

## 3.4. GPoCM: more complex manipulation method than FiLM (3)

- Since this channel-wise linear combination can also be viewed as a point-wise convolution, we name it PoCM. With inter-channel operations, PoCM can modulate features more flexibly and expressively than FiLM.

- Instaed of PoCM, we use Gated PoCM (GPoCM), since GPoCN is robust for source separation task. It is natural to use ***gated*** apporach the source separation tasks becuase a sparse latent vector (that contains many near-zero elements) obtained by applying GPoCMs, naturally generates separated result (i.e. more silent than the original).
- $GPoCM(X^{i}_{c}|\omega_{c}^{i},\beta_{c}^{i}) = \sigma(PoCM(X^{i}_{c}|\omega_{c}^{i},\beta_{c}^{i})) \odot X^{i}_{c}$
  - where $\sigma$ is a sigmoid and $\odot$ means the Hadamard product. 

---


## Experimental Results

![width:1200](https://imgur.com/GYaO0Aa.png)

---

## LaSAFT + GPoCM

- achieved [state-of-the-art](https://paperswithcode.com/sota/music-source-separation-on-musdb18?p=lasaft-latent-source-attentive-frequency) SDR performance on vocals and other tasks in Musdb18.

![width:900](https://imgur.com/A7JpmD9.png)
- news: outdated :(

---

## Discussion

- The authors of cunet tried to manipulate latent space in the encoder, 
  - assuming the decoder can perform as a general spectrogram generator, which is `shared' by different sources.

- However, we found that this approach is not practical since it makes the latent space (i.e., the decoder's input feature space) more discontinuous. 

- Via preliminary experiments, we observed that applying FiLMs in the decoder was consistently better than applying FilMs in the encoder.

---

## Links

- Choi, Woosung, et al. "Investigating u-nets with various intermediate blocks for spectrogram-based singing voice separation." 21th International Society for Music Information Retrieval Conference, ISMIR, Ed. 2020.
  - [Abstract, Paper, Poster, and Video](https://program.ismir2020.net/poster_2-04.html)
  - [Github](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS)
  
- Choi, Woosung, et al. "LaSAFT: Latent Source Attentive Frequency Transformation for Conditioned Source Separation." arXiv preprint arXiv:2010.11631 (2020).
  - [ArXiv](https://arxiv.org/abs/2010.11631)
  - [Github](https://github.com/ws-choi/Conditioned-Source-Separation-LaSAFT)
  - [Demo tracks](http://lasaft.github.io/)
