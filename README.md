# **CycleGAN** with **Differentiable Augmentation** for Style Transfer

As a reference, here is the main CycleGAN paper: JY Zhu et al "Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networks". 
Link: https://arxiv.org/pdf/1703.10593.pdf

and the Differential Augmentation Paper: S Zhao et al "Differentiable Augmentation for Data-Efficient GAN Training".
Link: https://arxiv.org/pdf/2006.10738.pdf

The training dataset consists of Monet paintings and photos that need to be converted into Monet-style paintings. The two datasets are not paired.

## CycleGAN:

Here, we need to translate images from a source domain X (that of original photos) to a target domain Y (that of Monet paintings).

Very simply put, a vanilla GAN consists of a Generator that tries to generate images of our required type, and a Discriminator that is fed in real images from our training set and images generated by the generator and tries to distinguish between the real and fake images. The Generator has to 'fool' the Discriminator by generating images so close to the real image distribution that the Discriminator labels these fake images as real ones.

In this task, however, we need to learn a mapping G from domain X->domain Y. Just this one constraint doesn't give us a unique mapping function-- there are infinitely many mappings G that map X to Y. Moreover, this also doesn't give robustness against mode collapse. In order to constrain our system furthur, we need to introduce **cycle consistency**, which exploits the fact that translation needs to be cycle consistent-- i.e., an image mapped from X->Y and back should yield the same image. 

Therefore we have 2 generators Gx mapping from X->Y and Gy mapping from Y->X. We also need 2 adverserial discriminators-- Dx which encourages Gx to translate an image from domain X into an image having a distribution extremely close to domain Y and Dy that does the same while translating the image from domain Y to domain X.

**Generator**: The Generator used in this repository is a UNet with skip connections. We have a monet generator (photo->Monet) and a photo generator (Monet->photo).

**Discriminator**: The Discriminator used in this repository is a PatchGAN type discriminator that outputs a 30x30 image. Each 30x30 patch classifies a 70x70 patch as real or fake. Higher pixel values indicate a real classification and lower values indicate a fake classification.

![](https://github.com/shreshtashetty/CycleGANPhototoMonet/blob/main/CycleGAN.PNG) [1]

##

**Loss Functions**:

***Generator loss function***:The generator wants to fool the discriminator into thinking the generated image is real. The perfect generator will have the discriminator output only 1s. Thus, it compares the generated image to a matrix of 1s to find the loss.

***Discriminator loss function***: The discriminator loss function compares real images to a matrix of 1s and fake images to a matrix of 0s. The perfect discriminator will output all 1s for real images and all 0s for fake images. The discriminator loss outputs the average of the real and generated loss.

***Cycle consistency loss function***: We want our original photo and the twice transformed photo to be similar to one another. Thus, we can calculate the cycle consistency loss by finding the average of their difference.

***Identity loss function***: The identity loss compares the image with its generator (i.e. monet with monet generator and photo with photo generator). If given a photo as input, we want it to generate the same image as the image was originally a photo. The identity loss compares the input with the output of the generator.

##

**Evaluation Metric**:

***Frechet Inception Distance (FID)***: 
The FID metric is the squared Wasserstein metric between two multidimensional Gaussian distributions-- the distribution of some neural network features of the images generated by the GAN and the distribution of the same neural network features from the "world" or real images used to train the GAN. As a neural network the Inception v3 trained on the ImageNet is commonly used. As a result, it can be computed from the mean and the covariance of the activations when the synthesized and real images are fed into the Inception network as follows:

![](https://wikimedia.org/api/rest_v1/media/math/render/svg/d2400530e03017712e6e21ccf71922dd15d052b0) 

##

**Additional Details**:

***Differential Augmentation***: The dataset being used for training had 300 Monet images and 7308 photos. Since there was very little Monet image data, it lead to the model overfitting. This called for the use of some kind of augmentation.
Differential Augmentation applies an augmentation to the real images as well as the generated images before they are sent into the discriminator. In order to update the generator, the gradients flow back through the discriminator and the 'augmentation function', hence this augmentation needs to be differentiable. Refer to the paper for a more in depth explanation.

![](https://github.com/shreshtashetty/CycleGANPhototoMonet/blob/main/DiffAugment.PNG) [2]

***Dataset used***: Images for training and testing are obtained from Kaggle. Their learning competition on CycleGANs-- "I am Something of a Painter Myself" has the entire dataset that can be downloaded as a .zip file. It has tfrecords too, which one can directly work with. 

## 

## Citations

[1] @inproceedings{CycleGAN2017,
  title={Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networkss},
  author={Zhu, Jun-Yan and Park, Taesung and Isola, Phillip and Efros, Alexei A},
  booktitle={Computer Vision (ICCV), 2017 IEEE International Conference on},
  year={2017}
}

[2] @inproceedings{zhao2020diffaugment,
  title={Differentiable Augmentation for Data-Efficient GAN Training},
  author={Zhao, Shengyu and Liu, Zhijian and Lin, Ji and Zhu, Jun-Yan and Han, Song},
  booktitle={Conference on Neural Information Processing Systems (NeurIPS)},
  year={2020}
}

