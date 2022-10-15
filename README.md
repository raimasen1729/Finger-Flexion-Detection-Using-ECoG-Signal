# Finger Flexion Detection using ECoG signals

The goal of this project was to predict finger flexion motions of individual fingers from Electrocorticography (ECoG) data recorded from three subjects. To achieve this, our team adopted a Deep Learning approach. Our final algorithm consists of a collection of Neural Networks to make finger-wise predictions. The ECoG data for each subject was first pre-processed using an FIR Band-pass filter, followed by a Notch filter. Next, eight essential (time-domain and frequency-domain) features were identified and collected over the ECoG data using moving windows of length 100ms and overlap 50ms. The feature set was then used for training the models to enable predictions on unknown test data. Finally, the resulting predictions were interpolated using Cubic Spline and post-processed by convolution with a Trapezoid Kernel to improve the correlation between our predictions and the actual values.

## Flexion predictions before and after post processing for one subject
<img src="https://user-images.githubusercontent.com/46754269/196006761-cd75afa7-b7a9-400e-8c9b-404a4e56373f.png" width="800" height="300"> 

## Flow Chart
<img src="https://user-images.githubusercontent.com/46754269/196006808-250b36dd-3441-4c53-8d7a-7fb18d391e93.png" width="400" height="500"> 

## Reference
> Decoding Flexion of Individual Fingers using Electrocorticographic Signals in Humans by Kubanek et al. (2009).
> Decoding Visual Information From a Population of Retinal Ganglion Cells by Warland et al. (1997).
> Decoding Finger Flexion from Band-Specific ECoG Signals in Humans by Nanying Liang and Laurent Bougrain (2012).
