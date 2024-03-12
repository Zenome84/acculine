# AccuLine ECG Noise Detection Assignment
This repository is an assignment for the Data Science position at AccuLine

All code was run on a Windows 10 machine, in a Jupyter notebook running Python 3.8.5

Below summarizes findings and results, which can be found in this repository

# Stage 1 - Designing a Model to Detect Noise
The raw data was downloaded from [CINC 2017](https://physionet.org/content/challenge-2017/1.0.0/#figure1) and [CINC 2011](https://physionet.org/content/challenge-2011/1.0.0/) datasets, with the latter using 'set-a'.

Both datasets came with annotations of "noisy" or otherwise ECG data. Upon manual inspection of several samples, it is clear that the "non-noisy" data did have several noise artifacts in some places, while the noisy data had some, possibly, good signals. I make note that "noise" in this context is not just static noise, but any kind of signal that isn't deemed useful or true enough to the actual heartbeat.

To construct a **Noise Detector**:
* I created subset signals from each signal with the assumption that we are likely to get clean subset signals from clean signals and noisy subset signals from noisy signals.
* A window of 600 was used, which corresponds to having a full 30 bpm beat if the signal was sampled at 300 Hz - 'set-a' was downsampled from 500 Hz to have uniform sampling.
* Two NN classifiers approaches are used:

    1. LSTM classifier on the raw signal windows (see [ecgV1.ipynb](./ecgV1.ipynb)).
    2. LSTM classifier on the Frequency domain - specifically using the DC and Spectral Entropy as two features into the LSTM (see [ecgV2.ipynb](./ecgV2.ipynb)).

* More details can be found in each respective Jupyter notebook.
* Ultimately, approach 1 was picked because it was resistant to over-fitting, and the results on the train/valid/test data were very similar.
* Hyperparameter tuning was done manually at the same time as picking different architectures before settling with this one (LSTM).

# Stage 2 - Marking Noisy Segments
Model 2 from Stage 1 was used on several segments from the test and shows that it is able to detect noise in as small a range as 600 samples, or, 2 seconds for 300 Hz. This can be found in [noisySegments.ipynb](./noisySegments.ipynb)

Most signals that were classified as noisy were either fully or almost fully marked as noisy. Furthermore, in the clean signals that had some noise, the model was able to detect those noisy sections.

# Additional Notes and Further Investigation

**On Choosing a NN Approach:** Prior to choosing this approach, I started reading about ECG analysis, and the PRQST wave shape. While I generally prefer to design/extract features from signals and then conduct classification using those features, this direction would require more time without prior domain knowledge. As such, the use of NN provides a good quick-turnaround approach, especially when the results provide a good baseline for further improvement later.

**Variational Autoencoder:** I also contructed a VAE model to perform dimensionality reduction on the signals to identify anomalous signals. The work for this can be found in [vae.ipynb](vae.ipynb) and [noisySegmentsVAE.ipynb](noisySegmentsVAE.ipynb). This approach only uses clean signals to discover a latent representation of valid ECG beats. As a result, a good approach to finding noise/anomalies is to pass a subsignal through it and then if the MSE of the output with the input is above a particular threshold, then the segment is classified as noise. I left it out of the main submission due to time, but offer it as a possible continuation because the initial results were promising.