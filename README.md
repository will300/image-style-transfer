# Image Style Transfer

## Project Description
This repository houses a project using Convolutional Neural Networks to transfer the style of a reference image on to a separate image, while maintaining the structure of the second image. The final model is a UNet which accepts any image as input, and transfers the style of a specific separate image (in the example given this is a painting by Van Gogh) to the output. The training takes roughly 30 minutes per style image on a Colab Pro GPU, and can only use one image as the style reference.

The loss function for the optimisation process, inspired by this paper https://rn-unison.github.io/articulos/style_transfer.pdf, is a weighted sum of the style difference between the output and the style image, and the structural difference between the output and input. 

The difference in style is calculated by passing both the the output and the style image through four layers of a separate VGG-Net and calculating a set of "Grams" on the output of each feature map stack, to which an MSE error is applied. The grams are calculated by taking the dot product between each possible combination of feature map in a stack, outputting a 1D array. 

The structural difference is calculated on the MSE of the fourth layer output from the VGG. Being one of the final layers of the network, this layer contains  most of the high-level structural information and spatial arrangement of the image. This is in opposition to the style evaluation which taken from a range of layers to capture information ranging from fine-detail to high-level. There is also no need to calculate the grams for the structural loss as this operation removes spatial relationships between different areas of the image, somthing required for stylistic but not structural analysis.

The model used for this implementation was a UNet (https://arxiv.org/abs/1505.04597), originally proposed as a tool for segmentation, using the FastAI image processing library. The UNet architecture is essentially an autoencoder with skip connections between layers that mirror each other on the encoder and decoder sides. This implementation removes the longest of these skips connections as passing the raw input to the final layer was found to influence the output too strongly.

While the model doesn't achieve such stunning results as the paper mentioned as inspiration for the project, this was never the intention. What this model can do that the paper can't, is provide a means to rapidly transfer any given style on to a large data set of images in a reasonable time. Once the model is trained to transfer style, the only time needed for each style transfer is a single forward pass of the network. This is in contrast to the paper, where each individual style transfer must be optimised separately and takes minutes or even hours.

## Setup Instructions
The project can be trained and tested using the step-by-step cells in the Google Colab Notebook. Unless using an offline GPU, it is recommended to connect to one using Colab. The training process first primes the model as a standard autoencoder, reproducing the image shown to it. The full style transfer training is done afterwards.

* Add a folder to the home directory in Google Drive named data
* Within this add a folder called images containing a small number of jpeg images (~50 should be sufficient)
* Choose a style image, the higher quality the better. The original model used a .tif image (raw uncompressed file format too large for uploading here). The jpeg provided may not be of high enough quality to produce worthwhile results
* First, work through the notebook to running the cells used to prime the autoencoder and train this for ~60 epochs (this should take about 10 minutes using a small 50 image data set)
* Then, run the cells related to training the full model, including adding the style transfer head, load the autoencoder weights into the UNet body and train the full model for ~120 epochs (can take about 2 hours)
* Once the autoencoder has been trained and its weights saved, this part of the network can be reused when training new styles
