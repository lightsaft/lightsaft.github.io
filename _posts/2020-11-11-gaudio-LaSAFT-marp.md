---
marp: true
---

# LaSAFT: Latent Source Attentive Frequency Transformation for Conditioned Source Separation

### Woosung Choi, Minseok Kim, Jaehwa Chung, and Soonyoung Jung

#### Our code and models are available [online](https://github.com/ws-choi/Conditioned-Source-Separation-LaSAFT).
##### You can also check separated samples from [here](https://lasaft.github.io).

[opening bgm](/slide/gaudio/music/)

---

## Contents

### 1. Task Definition: *Conditioned* Source Separation
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
    - Periodic sounds are typically comprised of a fundamental frequency and a number of harmonics which are spaced apart by relationships dictated by the source of the sound. It is the mixture of these harmonics that determines the timbre of the sound.
  ![width:300](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Spectrogram_of_violin.png/642px-Spectrogram_of_violin.png)

---

## 2.2. What's wrong with CNNs and spectrograms for audio processing?

- Yin, Dacheng, et al."**PHASEN**: A Phase-and-Harmonics-Aware Speech Enhancement Network." AAAI. 2020.
  > Non-local correlations exist in a T-F spectrogram along the
frequency axis. A typical example is the correlations among harmonics ... However, simply stacking several 2D convolution layers with small kernels cannot capture such global correlation. 
- Park, Soochul, and Ben Sangbae Chon. "GSEP: A robust vocal and accompaniment separation system using gated CBHG module and loudness normalization." arXiv preprint arXiv:2010.12139 (2020).
  > The two-dimensional convolution network used in the Spleeter for a frequency component misses the useful information at lower or higher frequency components which is out of the kernel range.
  
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
`"

---

## 2.3. Injecting TDFs into a U-Net framework

- Building Block TFC-TDF: Densely connected 2-d Conv (TFC) with TDFs

  ![width:700](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS/raw/cc6fb8048da2af3748aece6ae639af51b93c0a84/paper_with_code/img/tfctdf.png)

- U-Net with TFC-TDFs

  ![width:300](https://imgur.com/tmhMpqL.png)   + ![width:500](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS/raw/cc6fb8048da2af3748aece6ae639af51b93c0a84/paper_with_code/img/tfctdf.png)

---

## 2.3. Results?

![](https://imgur.com/Dby4Rkw.png)

---

## 2.3. Why does it work?: Weight visualization


![width:500](https://imgur.com/bH4cgKq.png) ![width:500](https://imgur.com/8fSULqe.png)


---

### 3. Part 2: LaSAFT for Conditioned Source Separation

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

![](https://imgur.com/A7JpmD9.png)

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
