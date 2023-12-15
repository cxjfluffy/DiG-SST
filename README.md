# DiG-SST

Welcome to the official repository for the DiG-SST method. This repository is dedicated to providing detailed information and code related to the DiG-SST method.

## Introduction

**DiG-SST (Divergence-Guided Simultaneous Speech Translation)** focuses on enhancing both translation quality and latency for streaming input. At its core, DiG-SST is a tightly integrated method specifically designed for the challenges of real-time speech translation.

Our approach introduces a simple yet effective prefix-based strategy for training translation models with partial speech input. This allows for more accurate and context-aware translations even with incomplete input, a common scenario in real-time applications.

Furthermore, DiG-SST incorporates an adaptive policy that makes informed read/write decisions for the translation model. This policy is based on anticipating the expected divergence in translation distributions resulting from future input, thereby optimizing for both accuracy and speed.

Extensive experiments conducted on multiple translation directions of the MuST-C benchmark have demonstrated that DiG-SST achieves a superior trade-off between translation quality and latency compared to existing methods. This makes it a promising solution for applications requiring high-quality real-time translation.


## More Details
### Montreal Forced Aligner
In the MuST-C dataset, some audio samples only contain music or applause, which do not provide valuable information for simultaneous speech translation scenarios. Therefore, we employ the [Montreal Forced Aligner (MFA)](https://github.com/MontrealCorpusTools/Montreal-Forced-Aligner) to process the ASR data and obtain speech-text alignment information, thereby filtering out those samples that cannot be aligned using MFA.

### Data Statistics after MFA Filtering
Below is a table showing the sentence counts for the MuST-C En-De, En-Es, and En-Fr datasets after applying Montreal Forced Aligner (MFA) filters:

| Split        | En-De | En-Es | En-Fr |
|--------------|-------|-------|-------|
| Train        | 217K  | 250K  | 259K  |
| Dev          | 1317  | 1214  | 1303  |
| Tst-COMMON   | 2490  | 2350  | 2482  |

*Table: Sentence counts for MuST-C En-De, En-Es, and En-Fr datasets after applying Montreal Forced Aligner (MFA) filters.*

We employed the MFA-filtered datasets for the training of both our offline and online models. To ensure a fair comparison, all experimental outcomes, for both the online and offline setups, were assessed using the unfiltered tst-COMMON set in its entirety.


### Intuition of the Prefix-Enhanced Translation
The approach of prefix-enhanced translation is inspired by SimulMT's multi-path wait-$k$ training to maintain consistency between training and inference. During inference, the model makes predictions based on incomplete audio and partial translations. Prefix tuning effectively bridges this gap, simplifying the process by eliminating the need to determine word boundaries in audio during training. Instead, we leverage prefix tuning in combination with a proficient policy model for read/write decisions, enhancing translation quality.

### Training Settings
In the initial stage of our training process, we incorporated the output data of the offline machine translation model, as shown in Figure [2-structure](a), as knowledge distillation data. We trained every part of our model for 40 epochs, implementing an early stopping mechanism if no improvement was observed for 20 epochs. 

The second stage of training focused on the policy module, which lasted for 10 epochs, with other components of the model remaining frozen. We experimented with the number of transformer layers in the policy module, ranging from 1 to 6. The optimal configuration was found to be 3 layers. A reduction in the number of layers to 1 or 2 resulted in a slight degradation in quality (by approximately 0.2 BLEU points), and no significant improvement was noted with more than 3 layers.

### Inference Process of DiG-SST

The inference process of DiG-SST is detailed in Algorithm 1 . The process involves a series of read and write actions:

- **Read Action**: Based on the existing partial audio, the model will additionally load the subsequent 100ms audio segment.
- **Wait-k Policy**: In the wait-k policy, the wait count (`k`) is measured in 300ms audio segments. For each read action, as the model reads a subsequent 100ms audio segment, `k` increases by 1/3, reflecting the proportional part of the 300ms segment.
- **Initial Delay**: There is an initial 600ms delay before starting the audio processing to address potential empty frames at the beginning of the samples.
- **Control Parameters**: Both `k` (as in "wait-k") and `λ` control the trade-off between quality and latency. This trade-off is discussed in the subsection "Why necessary to combine with wait-k" and depicted in Figures 5 and 6. The values of `k` and `λ` are selected empirically to achieve a desired trade-off. Regarding `λ` specifically, we suggest `λ` within the range [0, 0.15] (see Figure 6).
- With a fixed-length partial input `s_1:t`, our divergence-guided read/write policy will continuously generate output tokens as long as the write action is repeatedly triggered.

![Algorithm 1](https://github.com/cxjfluffy/DiG-SST/blob/main/Inference_process.png?raw=true)



## Installation
Coming soon.
