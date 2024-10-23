# [Unsupervised Denoising for Signal-Dependent and Row-Correlated Imaging Noise](https://arxiv.org/abs/2310.07887)<br>
## COSDD (COrrelated and Signal-Dependent Denoising)
<sup>1</sup>Benjamin Salmon and <sup>2</sup>Alexander Krull<br>
<sup>1, 2</sup>University of Birmingham<br>
<sup>1</sup>brs209@student.bham.ac.uk, <sup>2</sup>a.f.f.krull@bham.ac.uk<br>
This project includes code from the [ladder-vae-pytorch](https://github.com/addtt/ladder-vae-pytorch) project, which is licensed under the MIT License.


<img src="./resources/matrix.png">


Abstract: Accurate analysis of microscopy images is hindered by the presence of noise. This noise is usually signal-dependent and often additionally correlated along rows or columns of pixels. Current self- and unsupervised denoisers can address signal-dependent noise, but none can reliably remove noise that is also row- or column-correlated. Here, we present the first fully unsupervised deep learning-based denoiser capable of handling imaging noise that is row-correlated as well as signal-dependent. Our approach uses a Variational Autoencoder (VAE) with a specially designed autoregressive decoder. This decoder is capable of modeling row-correlated and signal-dependent noise but is incapable of independently modeling underlying clean signal. The VAE therefore produces latent variables containing only clean signal information, and these are mapped back into image space using a proposed second decoder network. Our method does not require a pre-trained noise model and can be trained from scratch using unpaired noisy data. We show that our approach achieves competitive results when applied to a range of different sensor types and imaging modalities.

## Getting started
### Environment
It is recommended to install the dependencies in a conda environment. If you haven't already, install miniconda on your system by following this [link](https://docs.conda.io/projects/miniconda/en/latest/miniconda-install.html).<br>
Once conda is installed, create and activate an environment by entering these lines into a command line interface:<br>
1. `conda create --name cosdd`
2. `conda activate cosdd`

Next, install PyTorch and torchvision for your system by following this [link](https://pytorch.org/get-started/locally/).<br> 
After that, you're ready to install the dependencies for this repository:<br>
`pip install lightning jupyterlab matplotlib tifffile scikit-learn scikit-image tensorboard`

### Tutorial notebooks
This repository contains three tutorial notebooks, training.ipynb, prediction.ipynb and generation.ipynb.

### Command line interface
COSDD takes a long time to train. It is recommended to train it in the terminal, instead of the notebooks.
Training and prediction configurations are set using .yaml files. An example training config file is `example-train-config.yaml` and an example prediction config file is `example-predict-config.yaml`. See below for options. 

To train, run:<br>
`python training.py example-train-config.yaml`

After training, use the model to denoise by running:<br>
`python prediction.py example-predict-config.yaml`

## Yaml training config file options

Important options are: `model_name`, `data: paths, patterns & axes`, `train-parameters: max-time`, `hyper-parameters: number-gaussians & noise-direction`. If training fails due to NaNs, see `data: clip-outliers`, `train-parameters: monte-carlo-kl`, `hyper-parameters: number-gaussians`.
<details>
      <summary>model-name</b></summary>

      (str) Name the trained model will be saved as.

</details>
<br>
data: 
<details>
      <summary>paths</summary>

      (str) Path to the directory the training data is stored in. Can be a list of strings if using more than one directory
      
</details>
<details>
      <summary>patterns</summary>

      (str) glob pattern to identify files within `paths` that will be used as training data. Current accepted file types are tiff/tif, czi & png. Edit get_imread_fn in utils.py to add data loading funtions for currently unsupported filetypes.
      
</details>
<details>
      <summary>axes</summary>

      (str) (S(ample) | C(hannel) | T(ime) | Z | Y | X). Describes the axes of the data as they are stored. I.e., when we call tifffile.imread("your-data.tiff"), what will be the shape of the returned numpy array? 
      The sample axis can be repeated, e.g. SCSZYX, if there are extra axes that should be concatenated as samples.
      
</details>
<details>
      <summary>number-dimensions</summary>

      (int) Number of spatial dimensions of your images. Default: 2.
      If your data has shape [T(ime), Y, X], the time dimension can be optionally treated as a spatial dimension and included in convolutions by setting this parameter to 3. If your data has shape Z, Y, X, the Z axis can be optionally treated as a sample dimension and excluded from convolutions by setting this parameter to 2.
      
</details>
<details>
      <summary>patch-size</summary>

      (list(int) | None) [(Depth), Height, Width]. Set to patch data into non-overlapping windows. Defualt: None.
      The training/validation split is made along the sample axis. If your data has only one sample, use this to break it into  patches that will be concatenated along the sample axis. 
      This is different from crop-size below, as it is deterministic and done once at the start of training.
      
</details>
<details>
      <summary>clip-outliers</summary>

      (bool) Hot or dead outlier pixels can disrupt training. Default: False.
      Set this to True to clip extreme pixel values between 1st and 99th percentile.
      
</details>
<br>
train-parameters:
<details>
      <summary>max-epochs</summary>

      (int) Maximum number of epochs to train. Default: 1000.
      
</details>
<details>
      <summary>max-time</summary>

      (str | None) Maximum time to train for. Default: None.
      Must be of form "DD:HH:MM:SS", or just `None`.
      COSDD can take a long time to converge, so use this to stop training in a reasonable time.
      
</details>
<details>
      <summary>patience</summary>

      (int) Stop training when validation loss plateaus for this number of epochs. Default: 100.
      
</details>
<details>
      <summary>batch-size</summary>

      (int) Number of images passed through network at a time. Default: 4.
      
</details>
<details>
      <summary>number-grad-batches</summary>

      (int) Gradient accumulation. Default: 4.
      Number of batches to pass through network before updating model parameters.
      
</details>
<details>
      <summary>crop-size</summary>

      (list(int))  [(Depth), Height, Width]. Default: [256, 256].
      As a form of data augmentation, at each training step a patch is randomly cropped from each training image. Set the size of that patch here.
      This is different from patch-size above as it is random and repeated at every training step.
      
</details>
<details>
      <summary>training-split</summary>

      (float) Percent of data to use as training set. Default: 0.9.
      1 - training-split is used as validation set.
      
</details>
<details>
      <summary>monte-carlo-kl</summary>

      (bool) Experimental. Default: False.
      Set True to calculate KL divergence using random samples from posterior. 
      I've found this can help when training crashes due to NaNs.
      Set False to calculate KL divergence analytically (common method).
      
</details>
<details>
      <summary>direct-denoiser-loss</summary>

      (str) "L1" or "MSE". Default: "MSE".
      Train direct denoiser to calculate coordinate-median or mean, respectively.
      
</details>
<details>
      <summary>use-direct-denoiser</summary>

      (bool) Train the direct denoiser to predict the average of samples. Default: True.
      Increases training time but reduces inference time. Worthwhile if inference is on a large dataset (GBs).
      
</details>
<br>
hyper-parameters:
<details>
      <summary>noise-direction</summary>

      (str) "x", "y" or "z". Default: "x".
      Axis along which noise is correlated.
      
</details>
<details>
      <summary>s-code-channels</summary>

      (int) Number of feature channels in the latent code describing the clean signal. Default: 64.
      Half of this value will be used as feature channels in VAE.
      
</details>
<details>
      <summary>number-layers</summary>

      (int) Number of levels in Ladder VAE. Default: 14.
      
</details>
<details>
      <summary>number-gaussians</summary>

      (int) Number of components in Gaussian mixture model used to model the noise. Default: 3.
      If noise is reproduced in output, increase this value. If training fails, reduce this value.
      
</details>
<br>
memory:
<details>
      <summary>precision</summary>

      (str) Floating point precision for training. Default: "bf16-mixed".
      "32-true"
      "32"
      "16-mixed"
      "bf16-mixed"
      "16-true"
      "bf16-true"
      "64-true"
      See https://lightning.ai/docs/pytorch/stable/common/precision.html

</details>
<details>
      <summary>checkpointed</summary>

      (bool) Whether to use activation checkpointing. Default: True.
      Set True to save GPU memory. Set False to increase training speed.

</details>
<details>
      <summary>gpu</summary>

      (list(int)) Index of which available GPU to use. Default: [0].

</details>

## Yaml prediction config file options
Important options are: `model_name`, `data: paths, patterns & axes`.
<details>
      <summary>model-name</b></summary>

      (str) Name of the trained model.

</details>
<br>
data: 
<details>
      <summary>paths</summary>

      (str) Path to the directory the data is stored in. Can be a list of strings if using more than one directory
      
</details>
<details>
      <summary>patterns</summary>

      (str) glob pattern to identify files within `paths` that will be used as prediction data. Current accepted file types are tiff/tif, czi & png. Edit get_imread_fn in utils.py to add data loading funtions for currently unsupported filetypes.
      
</details>
<details>
      <summary>axes</summary>

      (str) (S(ample) | C(hannel) | T(ime) | Z | Y | X). Describes the axes of the data as they are stored. I.e., when we call tifffile.imread("your-data.tiff"), what will be the shape of the returned numpy array? 
      The sample axis can be repeated, e.g. SCSZYX, if there are extra axes that should be concatenated as samples.
      
</details>
<details>
      <summary>number-dimensions</summary>

      (int) Number of spatial dimensions of your images. Default: 2.
      If your data has shape [T(ime), Y, X], the time dimension can be optionally treated as a spatial dimension and included in convolutions by setting this parameter to 3. If your data has shape Z, Y, X, the Z axis can be optionally treated as a sample dimension and excluded from convolutions by setting this parameter to 2.
      
</details>
<details>
      <summary>patch-size</summary>

      (list(int) | None) [(Depth), Height, Width]. Set to patch data into non-overlapping windows. Defualt: None.
      
</details>
<details>
      <summary>clip-outliers</summary>

      (bool) Hot or dead outlier pixels can disrupt prediction. Default: False.
      Set this to True to clip extreme pixel values between 1st and 99th percentile.
      
</details>
<br>
predict-parameters:
<details>
      <summary>batch-size</summary>

      (int) Number of images passed through network at a time. Default: 1.
      
</details>
<br>
memory:
<details>
      <summary>precision</summary>

      (str) Floating point precision for training. Default: "bf16-mixed".
      "32-true"
      "32"
      "16-mixed"
      "bf16-mixed"
      "16-true"
      "bf16-true"
      "64-true"
      See https://lightning.ai/docs/pytorch/stable/common/precision.html

</details>
<details>
      <summary>gpu</summary>

      (list(int)) Index of which available GPU to use. Default: [0].

</details>

## BibTeX
```
@misc{salmon2024unsuperviseddenoisingsignaldependentrowcorrelated,
      title={Unsupervised Denoising for Signal-Dependent and Row-Correlated Imaging Noise}, 
      author={Benjamin Salmon and Alexander Krull},
      year={2024},
      eprint={2310.07887},
      archivePrefix={arXiv},
      primaryClass={eess.IV},
      url={https://arxiv.org/abs/2310.07887}, 
}
```