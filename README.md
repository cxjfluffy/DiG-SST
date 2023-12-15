# DiG-SST

Welcome to the official repository for the DiG-SST method. This repository is dedicated to providing detailed information and code related to the DiG-SST method.

## Introduction

**DiG-SST (Divergence-Guided Simultaneous Speech Translation)** focuses on enhancing both translation quality and latency for streaming input. At its core, DiG-SST is a tightly integrated method specifically designed for the challenges of real-time speech translation.

Our approach introduces a simple yet effective prefix-based strategy for training translation models with partial speech input. This allows for more accurate and context-aware translations even with incomplete input, a common scenario in real-time applications.

Furthermore, DiG-SST incorporates an adaptive policy that makes informed read/write decisions for the translation model. This policy is based on anticipating the expected divergence in translation distributions resulting from future input, thereby optimizing for both accuracy and speed.

Extensive experiments conducted on multiple translation directions of the MuST-C benchmark have demonstrated that DiG-SST achieves a superior trade-off between translation quality and latency compared to existing methods. This makes it a promising solution for applications requiring high-quality real-time translation.


## More Details
### Details of Implementation
#### Montreal Forced Aligner
In the MuST-C dataset, some audio samples only contain music or applause, which do not provide valuable information for simultaneous speech translation scenarios. Therefore, we employ the [Montreal Forced Aligner (MFA)](https://github.com/MontrealCorpusTools/Montreal-Forced-Aligner) to process the ASR data and obtain speech-text alignment information, thereby filtering out those samples that cannot be aligned using MFA.

#### Data Statistics after MFA Filtering
Below is a table showing the sentence counts for the MuST-C En-De, En-Es, and En-Fr datasets after applying Montreal Forced Aligner (MFA) filters:

| Split        | En-De | En-Es | En-Fr |
|--------------|-------|-------|-------|
| Train        | 217K  | 250K  | 259K  |
| Dev          | 1317  | 1214  | 1303  |
| Tst-COMMON   | 2490  | 2350  | 2482  |

*Table: Sentence counts for MuST-C En-De, En-Es, and En-Fr datasets after applying Montreal Forced Aligner (MFA) filters.*

We employed the MFA-filtered datasets for the training of both our offline and online models. To ensure a fair comparison, all experimental outcomes, for both the online and offline setups, were assessed using the unfiltered tst-COMMON set in its entirety.


#### Intuition of the Prefix-Enhanced Translation
The approach of prefix-enhanced translation is inspired by SimulMT's multi-path wait-$k$ training to maintain consistency between training and inference. During inference, the model makes predictions based on incomplete audio and partial translations. Prefix tuning effectively bridges this gap, simplifying the process by eliminating the need to determine word boundaries in audio during training. Instead, we leverage prefix tuning in combination with a proficient policy model for read/write decisions, enhancing translation quality.

#### Training Settings
In the initial stage of our training process, we incorporated the output data of the offline machine translation model, as shown in Figure [2-structure](a), as knowledge distillation data. We trained every part of our model for 40 epochs, implementing an early stopping mechanism if no improvement was observed for 20 epochs. 

The second stage of training focused on the policy module, which lasted for 10 epochs, with other components of the model remaining frozen. We experimented with the number of transformer layers in the policy module, ranging from 1 to 6. The optimal configuration was found to be 3 layers. A reduction in the number of layers to 1 or 2 resulted in a slight degradation in quality (by approximately 0.2 BLEU points), and no significant improvement was noted with more than 3 layers.

### The Inference Process of The Proposed DiG-SST

#### Input
- Streaming speech input **s**.
- Threshold **λ=0.1** by default.
- The value of **k** in wait-k policy.

#### Output
- Target outputs **ŷ**=(ŷ1, ŷ2, ..., ŷJ)

#### Algorithm
```python
def main():
    # Initialize hidden states after the translation decoder h^dec=None,
    # target index j=1, wait-k index k=0, ŷ=<BOS>, current received speech ŝ=[].
    h_dec = None
    j = 1
    k = 0
    y_hat = "<BOS>"
    s_hat = []

    # Read
    while True:
        s_hat.extend(s.read())  # Read from streaming speech input
        update_h_dec_based_on(y_hat, s_hat)  # Update h_dec based on y_hat and s_hat
        delta_hat = update_delta_hat_based_on(h_dec)  # Update delta_hat
        k += 1/3

        # Write
        while (delta_hat < lambda) or (k >= k_wait) or (s_hat == s):
            # Conditions: predicted divergence score smaller than threshold,
            # or k reaches the k of wait-k policy, or full speech input received.
            y_j_hat = generate_y_j_based_on(h_dec)  # Generate y_j_hat
            y_hat.append(y_j_hat)

            # Translation ends
            if y_j_hat == "<EOS>":
                return y_hat

            update_h_dec_based_on(y_hat, s_hat)  # Update h_dec
            delta_hat = update_delta_hat_based_on(h_dec)  # Update delta_hat
            j += 1
            k -= 1




## Installation
Instructions to set up the DiG-SST environment and run the code:
