# Target Speaker Separation Project

This repository contains the implementation of a draft approach to target speaker separation combining an auxiliary feature. The work integrates pitch extraction into the task through two distinct strategies: concatenation and joint training. The following sections provide an overview of the model architecture, training strategies, and experimental setup used in this study. 

## Introduction

Target speaker separation (TSS) has made significant strides with the development of deep learning and speech and language processing (SLP) technologies. These advancements enable the extraction of a target speaker's voice in environments with multiple simultaneous speakers, which is critical for enhancing AI-driven conversational systems. Traditional TSS methods, such as VoiceFilter, Atss-Net, and spex++, have focused on improving various components within the encoder-separator-decoder framework, leading to improved accuracy and robustness.

<p align="center">
  <img src="img/encoder_sep_decoder.png" alt="Previous TSS Architecture">
  <br>
  <b>Figure 1:</b> The previous architecture for target speaker separation (TSS), which utilizes an encoder-separator-decoder framework.
</p>

Despite these advancements, there remains a need for more generalized and robust training strategies that can be applied across different model architectures. This is particularly important as AI becomes increasingly involved in real-time conversational interactions, where accurately distinguishing and separating speakers in complex auditory environments is essential.

In this work, we introduce an approach that leverages pitch—a fundamental attribute of speech—to enhance TSS performance. We propose a target speaker pitch extraction module designed to estimate the pitch of a target speaker directly from a mixture of multiple speakers' utterances. This approach is highly relevant for Speech and Language Processing (SLP) applications, where precise speaker-specific features are crucial for improving tasks such as speech recognition and speaker identification.

To integrate the extracted pitch information into the TSS framework, following two different training strategies are explored:
- **Concatenation Training**: Directly concatenating the extracted pitch with speaker embeddings before feeding them into the TSS model.
- **Joint Training**: Simultaneously training the pitch extraction and TSS models to optimize both processes together.

The baseline model, a small-scale RNNoise system, serves as the foundation for these strategies.

## Model Architecture

### Overview

The proposed architecture consists of three main components:
1. **Speaker Embedding Extraction Module**: Processes an enrollment utterance to generate a 128-dimensional speaker embedding.
2. **Target Extraction Module**: Inputs the magnitude of the mixed audio along with the target speaker embedding
3. **TSS Module**: Utilizes the mixed utterance, speaker embedding, and extracted pitch to estimate the voice of the target speaker.

### Training Strategies Incorporating Pitch Information

#### 1. Concatenation Method

In this approach, the target pitch extracted from the mixed audio is concatenated with the speaker embedding along the feature axis. The combined features are then input into the speech separation model. Both the speaker embedding extraction and pitch extraction modules are pre-trained and kept fixed during TSS model training.

#### 2. Joint Training Method

This method involves simultaneous optimization of the pitch extraction and TSS models. By training these models together, the pitch information is more effectively integrated into the speaker separation process, resulting in improved performance.

### Target Pitch Extraction Module

<p align="center">
  <img src="img/tp_model.png" alt="Target Pitch Extraction Module">
  <br>
  <b>Figure 2:</b> The target pitch extraction module, which takes the spectrogram of the mixed utterance and the target speaker embedding as inputs, and outputs the 1-dimensional pitch information for the target speaker.
</p>

The pitch extraction module is based on an LSTM architecture. It processes the spectrogram of the mixed utterance and the target speaker embedding, outputting a time series of pitch values (\( f_0 \)) for the target speaker. The ground-truth pitch values are derived using the RAPT algorithm applied to the pre-mixed clean audio, with the pitch range set between 60 and 404 Hz. This task is framed as a regression problem, where the L1 loss function is used to minimize the difference between the predicted and actual pitch values:

![extraction as regression problem and adopt L1 loss](img/image-2.png)

where \( f̂_0 \) is the predicted pitch and \( f_0 \) is the ground truth.

To evaluate the accuracy of the pitch extraction, we use the precision rate (PR), defined as:
![precision rate for evaluation](img/image-1.png)

where \( N_{0.05} \) is the number of frames where the predicted pitch deviates by less than 5% from the ground truth, and \( N \) is the total number of frames.

### RNNoise w/ hierarchic blocks

The  model serves as the backbone of our speech separation system and is a modification of the established RNNoise model, widely used in speech enhancement. The RNN architecture, shown in Figure 4, features multiple RNN blocks. Each block consists of a fully connected (FC) layer followed by an RNNoise-like module, designed to compress and process the features after concatenation of the speaker embedding and the magnitude spectrogram.

To improve the model's capacity to represent complex features, the RNN block is repeated four times. Additionally, cumulative layer normalization (cLN) is applied between these blocks to stabilize and enhance training efficiency. The Conv1D layer is employed to expedite the computation of the short-time Fourier transform (STFT), and only the magnitude spectrogram \( X \) of the mixed audio is fed into the model. The phase spectrogram \( P \) is utilized later in the pipeline to reconstruct the estimated target speech using a Conv-Trans1D layer, which performs an inverse STFT (iSTFT). This process is mathematically expressed as:

![perform inverse short-time Fourier transform to get the estimated target speech](img/image.png)

where the element-wise product of the mixed spectrogram and the estimated mask \( M \) is computed. The final output is the estimated target speech, with the training objective being the scale-invariant signal-to-noise ratio (SI-SNR).


<p align="center">
  <img src="img/mbrnn_pitch.png" alt="MBRNN Model Details with Mag Mask Net Details">
  <br>
  <b>Figure 3:</b> The detailed architecture of the MBRNN model, including the specifics of the Mag mask network used within it.
</p>

<p align="center">
  <img src="img/rnnBlock.png" alt="RNN in MBRNN Module">
  <br>
  <b>Figure 4:</b> The structure of an RNN block within the MBRNN module, which includes a fully connected layer followed by an RNNoise-like module.
  
</p>
