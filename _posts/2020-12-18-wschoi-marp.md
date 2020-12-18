---
marp: true
---

## Woosung Choi

#### Ph.D. Candidate, Department of Computer Science
#### College of Informatics, Korea University


#### Contents ---

1. Selected Publications
  1-1 **[ISMIR 2020]**
  1-2 **[LaSAFT]** :  [![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/lasaft-latent-source-attentive-frequency/music-source-separation-on-musdb18)](https://paperswithcode.com/sota/music-source-separation-on-musdb18?p=lasaft-latent-source-attentive-frequency)

2. Research Interests
3. Research Proposal
4. As a songwriter

---

## 1. Selected Publications

- 1-1 [ISMIR 2020] **Choi, Woosung.**, Kim, Minseok., Chung, Jaehwa., Lee, Daewon., and Jung, Soonyoung. "Investigating u-nets with various intermediate blocks for spectrogram-based singing voice separation." 21th International Society for Music Information Retrieval Conference, ISMIR, 2020. 

- 1-2. [LASAFT] **Choi, Woosung.**, Kim, Minseok., Chung, Jaehwa., and Jung, Soonyoung. "LaSAFT: Latent Source Attentive Frequency Transformation for Conditioned Source Separation." arXiv preprint arXiv:2010.11631 (2020).

---

## 1-1 [ISMIR 2020]

**Choi, Woosung.**, Kim, Minseok., Chung, Jaehwa., Lee, Daewon., and Jung, Soonyoung. "Investigating u-nets with various intermediate blocks for spectrogram-based singing voice separation." 21th International Society for Music Information Retrieval Conference, **[ISMIR](https://program.ismir2020.net/poster_2-04.html)**, 2020. ( [github](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS), [colab](https://colab.research.google.com/github/ws-choi/ISMIR2020_U_Nets_SVS/blob/master/colab_demo/TFC_TDF_Net_Large.ipynb))

- Injecting FT(Frequency Transformation) blocks can improve the performance of U-Nets for the spectrogram-based singing voice separation model.


  ![](https://imgur.com/irPUaeQ.png) 
  - Singing voice Separation on [MUSDB18](https://sigsep.github.io/datasets/musdb.html) dataset

---


## 1-1 [ISMIR 2020] Background: U-Net

- U-Net: an encoder-decoder structure with symmetric skip connections
  - These symmetric skip connections allow models to recover fine-grained details of the target object during decoding effectively.
  ![width:500](https://imgur.com/ua3qkU4.png)

- Originally proposed for ***Medical Image Segmentation***
  - can also be viewed as an Image-to-Image Translation
  - The original U-Net is fully *2-d convolutional*


---

## 1-1 [ISMIR 2020]. Motivation: Spectrogram $\neq$ Image

- [What's wrong with CNNs and spectrograms for audio processing?](https://towardsdatascience.com/whats-wrong-with-spectrograms-and-cnns-for-audio-processing-311377d7ccd)
  - The axes of spectrograms do not carry the same meaning
    - spatial invariance that **2D CNN**s provide might not perform as well
  - The spectral properties of sounds are non-local
    - Periodic sounds are typically comprised of a fundamental frequency and a number of **harmonics** which are spaced apart by relationships dictated by the source of the sound. It is the mixture of these harmonics that determines the timbre of the sound.
  ![width:300](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Spectrogram_of_violin.png/642px-Spectrogram_of_violin.png)

---

## 1-1 [ISMIR 2020]. Motivation: Spectrogram $\neq$ Image

- Yin, Dacheng, et al."**PHASEN**: A Phase-and-Harmonics-Aware Speech Enhancement Network." AAAI. 2020.
  > Non-local correlations exist in a T-F spectrogram along the
frequency axis. A typical example is the correlations among harmonics ... However, simply stacking several 2D convolution layers with small kernels cannot capture such global correlation. 
- Park, Soochul, and Ben Sangbae Chon. "GSEP: A robust vocal and accompaniment separation system using gated CBHG module and loudness normalization." arXiv preprint arXiv:2010.12139 (2020).
  > The two-dimensional convolution network used in the Spleeter for a frequency component misses the useful information at lower or higher frequency components which is out of the kernel range.
  
---


## 1-1 [ISMIR 2020]. Alternatives to avoid Fully-2d-Convs

- 1-D CNNs
  - Liu, Jen-Yu, and Yi-Hsuan Yang. "Dilated convolution with dilated GRU for music source separation." arXiv preprint arXiv:1906.01203 (2019).
- Dilated Convolutions
  - Takahashi, Naoya, and Yuki Mitsufuji. "D3Net: Densely connected multidilated DenseNet for music source separation." arXiv preprint arXiv:2010.01733 (2020).
- ***FTBs***: Frequency Transformation Blocks
  - **ours**, PHASEN
- RNNs: $\sim$ FTBs
  - Takahashi, Naoya, Nabarun Goswami, and Yuki Mitsufuji. "Mmdenselstm: An efficient combination of convolutional and recurrent neural networks for audio source separation." 2018 16th International Workshop on Acoustic Signal Enhancement (IWAENC). IEEE, 2018.

---

## 1-1 [ISMIR 2020]. Injecting FTBs into U-Nets

- ***FTBs***: Frequency Transformation Blocks

  - An FTB called Time-Distributed Fully-connected Layer ([TDF](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS/blob/master/paper_with_code/Paper%20with%20Code%20-%203.%20INTERMEDIATE%20BLOCKS.ipynb)):
    ![width:800](https://github.com/ws-choi/ISMIR2020_U_Nets_SVS/raw/cc6fb8048da2af3748aece6ae639af51b93c0a84/paper_with_code/img/tdf.png)

  - Choi, Woosung, et al. "Investigating u-nets with various intermediate blocks for spectrogram-based singing voice separation." 21th International Society for Music Information Retrieval Conference, ISMIR, Ed. 2020.

---

## 1-1 [ISMIR 2020]. Experimental Results

- Singing voice Separation on [MUSDB18](https://sigsep.github.io/datasets/musdb.html) dataset

- Ablation (n_fft = 2048)
  - U-Net with 17 TFC blocks: SDR 6.89dB
  - U-Net with 17 TFC-**TDF** blocks: SDR 7.12dB (+0.23 dB)

- Large Model (n_fft = 4096)
  ![](https://imgur.com/irPUaeQ.png)

---

## 1-1 [ISMIR 2020]. Why does it work?: Weight visualization

- freq patterns of different sources captured by TDFs, of FTBs

![width:500](https://imgur.com/bH4cgKq.png) ![width:500](https://imgur.com/8fSULqe.png)

- left: ours, right: phasen

---

## 1-2 [LaSAFT]: Latent Source Attentive Frequency Transformation for Conditioned Source Separation

[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/lasaft-latent-source-attentive-frequency/music-source-separation-on-musdb18)](https://paperswithcode.com/sota/music-source-separation-on-musdb18?p=lasaft-latent-source-attentive-frequency)


**Choi, Woosung.**, Kim, Minseok., Chung, Jaehwa., and Jung, Soonyoung. "[LaSAFT](https://paperswithcode.com/paper/lasaft-latent-source-attentive-frequency): Latent Source Attentive Frequency Transformation for Conditioned Source Separation." arXiv preprint arXiv:2010.11631 (2020).

- [demo](http://lasaft.github.io/)
- [github](https://github.com/ws-choi/Conditioned-Source-Separation-LaSAFT)
- [colab](https://colab.research.google.com/github/ws-choi/Conditioned-Source-Separation-LaSAFT/blob/main/colab_demo/LaSAFT_with_GPoCM_Stella_Jang_Example.ipynb)

---


## 1-2 [LaSAFT]. Background: Conditioned Source Separation

- Task Definition
  - Input: an input audio track $A$ and a one-hot encoding vector $C$ that specifies which instrument we want to separate
  - Output: separated track of the target instrument
- Method: Conditioning Learning
  - can separate different instruments with the aid of the **control mechanism**. 
  - Conditioned-U-Net (C-U-Net)
    - Meseguer-Brocal, Gabriel, and Geoffroy Peeters. "CONDITIONED-U-NET: INTRODUCING A CONTROL MECHANISM IN THE U-NET FOR MULTIPLE SOURCE SEPARATIONS." Proceedings of the 20th International Society for Music Information Retrieval Conference. 2019.

---

## 1-2 [LaSAFT]. Background: C-U-Net

- Conditioned-U-Net extends the U-Net by exploiting Feature-wise Linear Modulation (FiLM)

  ![width:700](https://github.com/gabolsgabs/cunet/raw/master/.markdown_images/overview.png)


---

## 1-2 [LaSAFT]. Background: an overview of C-U-Net

  ![width:1200](https://github.com/gabolsgabs/cunet/raw/master/.markdown_images/c-u-net.png)

---

## 1-2 [LaSAFT]. Naive Extention: Injecting FTBs into C-U-Net?

- Baseline C-U-Net + TFC-TDFs
  ![width:600](https://imgur.com/NpB5BpU.png)

- TFC vs TFC-TDF
  ![width:600](https://imgur.com/e6giOhG.png)

---

## 1-2 [LaSAFT]. Naive Extention: Above our expectation

- TFC vs TFC-TDF
  ![width:600](https://imgur.com/e6giOhG.png)

- Although it does improve SDR performance by capturing common frequency patterns observed across all instruments,
  - Merely injecting an FTB to a CUNet **does not inherit the spirit of FTBs**

- We propose the Latent Source-Attentive Frequency Transformation (LaSAFT), a novel frequency transformation block that can capture instrument-dependent frequency patterns by exploiting the scaled dot-product attention

---

## 1-2 [LaSAFT]. The Motivation

- Extending TDF to the Multi-Source Task
  - Naive Extension: MUX-like approach
    - A TDF for each instrument: $\mathcal{I}$ instrument => $\mathcal{I}$ TDFs
    ![](https://upload.wikimedia.org/wikipedia/commons/e/e0/Telephony_multiplexer_system.gif)

  - However, there are much more 'instruments' we have to consider in fact
    - female-classic-soprano, male-jazz-baritone ... $\in$ 'vocals' 
    - kick, snare, rimshot, hat(closed), tom-tom ... $\in$ 'drums'
    - contrabass, electronic, walking bass piano (boogie woogie) ... $\in$ 'bass'  

---

## 1-2 [LaSAFT]. Latent Source-attentive Frequency Transformation

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

## 1-2 [LaSAFT]. Extending TDF to the Multi-Source Task (1)

![width:600](https://imgur.com/vQNgttJ.png)

- duplicate $\mathcal{I}_L$ copies of the second layer of the TDF,  where $\mathcal{I}_L$ refers to the number of ***latent instruments***.
  - $\mathcal{I}_L$ is not necessarily the same as $\mathcal{I}$ for the sake of flexibility
- For the given frame $V\in \mathbb{R}^F$, we obtain the $\mathcal{I}_L$ latent instrument-dependent frequency-to-frequency correlations, denoted by $V'\in \mathbb{R}^{F \times \mathcal{I}_L}$.

---

## 1-2 [LaSAFT]. Extending TDF to the Multi-Source Task (2)

![width:600](https://imgur.com/vQNgttJ.png)

- The left side determines how much each ***latent source*** should be attended
- The LaSAFT takes as input  the instrument embedding $z_e \in \mathbb{R}^{1 \times E}$. 
- It has a learnable weight matrix $K\in \mathbb{R}^{ \mathcal{I}_L \times d_{k}}$, where we denote the dimension of each instrument's hidden representation by $d_{k}$.
- By applying a linear layer of size $d_{k}$ to $z_e$, we obtain $Q \in \mathbb{R}^{d_{k}}$.

---

## 1-2 [LaSAFT]. Extending TDF to the Multi-Source Task (3)

![width:600](https://imgur.com/vQNgttJ.png)

- We now can compute the output of the LaSAFT as follows:

  - $Attention(Q,K,V') = softmax(\frac{QK^{T}}{\sqrt{d_{k}}})V'$

- We apply a LaSAFT after each TFC in the encoder and after each Film/GPoCM layer in the decoder. We employ a skip connection for LaSAFT and TDF, as in TFC-TDF.

---

## 1-2 [LaSAFT]. Effects of employing LaSAFTs instead of TFC-TDFs

- Dataset: [Musdb18](https://sigsep.github.io/datasets/musdb.html)

  ![width:600](https://imgur.com/sPVDDzZ.png)


---

## 1-2 [LaSAFT]. More complex manipulation method than FiLM

- FiLM (left) vs PoCM (right)

![width:550](https://imgur.com/A3kAxVS.png) ![width:550](https://imgur.com/9A4otVA.png)

- PoCM is an extension of FiLM. 
  - while FiLM does not have inter-channel operations
  - PoCM has inter-channel operations

---

## 1-2 [LaSAFT]. More complex manipulation method than FiLM (2)

- PoCM is an extension of FiLM

  - $FiLM(X^{i}_{c}|\gamma_{c}^{i},\beta_{c}^{i}) =  \gamma_{c}^{i} \cdot X^{i}_{c} + \beta_{c}^{i}$

  - $PoCM(X^{i}_{c}|\omega_{c}^{i},\beta_{c}^{i}) = \beta_{c}^{i} + \sum_{j}{\omega_{cj}^{i} \cdot X^{i}_{j}}$
    - where $\gamma_{c}^{i}$ and $\beta_{c}^{i}$ are parameters generated by the condition generator, and $X^{i}$ is the output of the $i^{th}$ decoder's intermediate block, whose subscript refers to the $c^{th}$ channel of $X$

    ![width:500](https://imgur.com/9A4otVA.png)

---

## 1-2 [LaSAFT]. More complex manipulation method than FiLM (3)

- Since this channel-wise linear combination can also be viewed as a point-wise convolution, we name it PoCM. With inter-channel operations, PoCM can modulate features more flexibly and expressively than FiLM.

- Instaed of PoCM, we use Gated PoCM (GPoCM), since GPoCN is robust for source separation task. It is natural to use ***gated*** apporach the source separation tasks becuase a sparse latent vector (that contains many near-zero elements) obtained by applying GPoCMs, naturally generates separated result (i.e. more silent than the original).
- $GPoCM(X^{i}_{c}|\omega_{c}^{i},\beta_{c}^{i}) = \sigma(PoCM(X^{i}_{c}|\omega_{c}^{i},\beta_{c}^{i})) \odot X^{i}_{c}$
  - where $\sigma$ is a sigmoid and $\odot$ means the Hadamard product. 

---

## 1-2 [LaSAFT]. Experimental Results: LaSAFT + GPoCM

[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/lasaft-latent-source-attentive-frequency/music-source-separation-on-musdb18)](https://paperswithcode.com/sota/music-source-separation-on-musdb18?p=lasaft-latent-source-attentive-frequency)


- achieved [state-of-the-art](https://paperswithcode.com/sota/music-source-separation-on-musdb18?p=lasaft-latent-source-attentive-frequency) SDR performance on vocals and other tasks in Musdb18.

![width:1200](https://imgur.com/GYaO0Aa.png)


---

## Discussion

- The authors of cunet tried to manipulate latent space in the encoder, 
  - assuming the decoder can perform as a general spectrogram generator, which is `shared' by different sources.

- However, we found that this approach is not practical since it makes the latent space (i.e., the decoder's input feature space) more discontinuous. 

- Via preliminary experiments, we observed that applying FiLMs in the decoder was consistently better than applying FilMs in the encoder.

---

## 2. Research Interest

I am currently interested in the following areas:

- Machine Learning-based Audio Editing
- Audio Source Separation
- Automatic Audio Mixing

---

## 3. Research Proposal

### Machine Learning-based Audio Editing for a user-friendly interface

Since I already started writing a paper for this project, I cannot share more information about it in detail, but I am currently working on a personal project called Machine Learning-based Audio Editing for a user-friendly interface. The goal of this research is to create an audio manipulation model equipped with a convenient user interface. I believe this project's result will be widely used in various audio signal processing software such as DAWs, or DAW-plugins, as many users have loved the ML-based applications of izotope. Decreasing the difficulty of audio editing will make more users create, edit, manipulate, and share their audio files. 

---

## 4. As a songwriter

Although I am not writing songs these days, I used to write and sing my songs.
The experiences give me some inspiration about future research topics as a DAW 'user'.

- My Original songs: [link](https://soundcloud.com/choi-hn/sets/cqro6jqcz6ox)
- Cover songs (, where I made the instrumental): [link](https://soundcloud.com/choi-hn/sets/rmx-by-choi-hn)