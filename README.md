# Finger-Flexion-Detection-Using-ECoG-Signal


1. SummaryoftheFinalAlgorithm
The goal of the Final Project was to predict finger flexion motions of individual fingers from Electrocorticography (ECoG) data recorded from three subjects. To achieve this, our team adopted a Deep Learning approach. Our final algorithm consists of a collection of Neural Networks to make finger-wise predictions. The ECoG data for each subject was first pre-processed using an FIR Band-pass filter, followed by a Notch filter. Next, eight essential (time-domain and frequency-domain) features were identified and collected over the ECoG data using moving windows of length 100ms and overlap 50ms. The feature set was then used for training the models to enable predictions on unknown test data. Finally, the resulting predictions were interpolated using Cubic Spline and post-processed by convolution with a Trapezoid Kernel to improve the correlation between our predictions and the actual values.
2. DetailedExplanationoftheFinalAlgorithm
A) Data
The training data for subjects 1, 2 and 3 had 62, 48 and 64 channels, respectively, with 300000 samples for each channel and was sampled at 1000 Hz. The finger flexions for this ECoG data which were recorded by the data-glove were sampled at 25 Hz. The testing data (leaderboard data) had the same number of channels as before for each subject but with a few samples (147500).
We first plotted all the channels for each of the three subjects to identify the erroneous channels. As a result, channel 54 from the first subject's data and channels 20 and 37 from the second subject's data were excluded. The third subject had no erroneous channels, so all the data was used.
B) Pre-processing
At each time point, the train and test signal was filtered in a 2-step manner with the first filtering done with Kaiser filter, and the second using Notch filter. To decide on a filter, it was first important to view the spectral composition of the signal to identify the noise frequencies. On performing spectral decomposition using fourier series conversion, the following type of chart was observed:
  
  The x-axis depicts the frequencies (in Hz) and the y-axis the amplitude. The values after 200 Hz can be ignored as the max required frequency was taken to be 200 Hz. The peaks before 200 Hz and after 0 Hz signify noise frequencies. The values turned out to be 60, 120 and 180 Hz. This makes sense as the electrodes used to measure the signals had an alternating frequency of 60 Hz. Therefore, it was decided to create a notch filter to filter out the noise frequencies. Aside from this, it was mentioned earlier that we had determined only the frequencies below 200Hz to be useful from literature surveys. In order to filter out the noise for frequencies above 200Hz, it was decided to use an FIR filter owing to the simplicity of the filter and versatility. For the window design, Kaiser method was chosen due to the independence of the method from window length and how the ripple size is easier to work around. Setting a ripple size of 60 decibels and transition width of 5Hz, we used a band pass filter for 4Hz-200Hz to design the filter.
C) Feature Extraction
For each channel of a particular subject in the train ECoG signal, the number of windows were first identified. Each window was of length 100ms and had an overlap of 50ms. Using the right-to-left convention, features were calculated over each window. We narrowed down to 8 important features for each channel - Average voltage, Line length, Energy, Average frequency in bins [5-15Hz], [20-25Hz], [75-115Hz], [125-160Hz] and [160-175Hz]. These bins are chosen in reference to the research paper by Kubanek et al[1]. Essentially, several bands of frequency ranging from 5-200 Hz were selected for analysis but the 35-70Hz band was omitted due to poor correlation. So for each subject we had a set of (channels x 8) features over all the windows. We then reversed the feature set to ensure it aligned with the original dataglove time sequence and NOT the right to left convention that was adopted while feature calculation. This was a crucial step in further assessment of predictions.
It was also necessary to downsample the data glove signal (train labels) to align with the same number of windows as in the train data for each subject. For the downsampling, a similar moving windows method was used, and the maximum amplitude was selected in each window. Since the sampling rate for the dataglove signal was 25Hz, we chose a window length of 6 seconds with an overlap of 2 seconds.

 Next, from the feature set, we created the response matrix R as described in Warland et al., 1997. Each column in the R matrix represents a feature or predictor. We chose to ‘look-back’ 3 time windows which resulted in 1465 columns or features across all channels for each time-window for each subject. We chose to use the R matrix as our feature set / input to the Neural Network models because of its ability to act as a ‘memory’ feature vector by taking into account features in the previous time windows.
D) Neural Network Architecture
The network architecture over all 3 subjects was kept consistent. The input data (R matrix) was initially scaled using standard scaling (removing mean and scaling to unit variance). This type of scaling proved to be better than Robust scaler and MinMax scaler options available in sklearn preprocessing scaling packages. This transformed data was passed into the neural network consisting of 4 layers. The first layer has 128 nodes, and the subsequent layers have 256 nodes. The final output layer has 1 node. The activation function was set to rectified linear unit as it is the most commonly used in most of the current literature and it performs exceptionally well in the vanishing gradient problem, which time series data is very prone to facing. The last layer’s activation function was set to linear as this was a neural network performing a regression task and not a classification one.
We chose the Adam Optimizer as its learning rate is adaptive. Our performance metric for this experiment (also called cost function/ loss function) was the mean absolute error. This was an unconventional choice as regression methods usually minimize the mean squared error. But this performance metric worked better for our problem. To prevent overfitting on the data, we adopted early stopping. While training over 500 epochs, the early stopping parameter would see when the validation loss starts increasing and train only for that many epochs. Apart from this, to maintain robustness of the model, a batch size of 32 samples were passed into the training each time for fingers 1, 2, and 4. For fingers 3 and 5, a batch size of 64 performed better. Using this architecture, predictions were made separately over all fingers.
E) Interpolation and Post-processing
To bring each of the predictions to the same size as the test data, interpolation was performed on the predictions using Cubic Spline. To ensure the end points remained the same, we set the method of Cubic Spline as ‘clamped’.
The resulting signal had a lot of noise and did not have the same pattern as the train dataglove signals. To reduce noise, the interpolated predictions were convolved with Trapezoid Kernels. The height and width of the kernels were identified through trial-and-error. Post-processing the interpolated predictions significantly improved the correlation.

    Kernel For Subject 1 Kernel For Subject 2 Kernel For Subject 3 A few plots showing the predictions before and after post-processing :
 Subject 2, Finger 1
 Subject 3, Finger 1
 Subject1, Finger 1
The resulting final predictions were then saved in MAT file as a 3x1 array where each row represented a
subject and was in the shape 147500 x 5 (for the leaderboard data).

 3. Flow Chart Summarizing the General Steps
4. Feature set (and eventually) R matrix reversal
The important change we made along the way was reversing the order of the feature set, since the calculation was done in the conventional right to left approach. It was necessary to reverse the order because although we were also downsampling the dataglove readings in the right-to-left manner (which acted as the train labels), the test labels were still aligned from left to right. Since it is not appropriate to modify the test labels in any form (including preprocessing), we soon discovered that bringing the feature set back to the left-to-right order (time series data) seemed more fitting to make predictions accurately. The following plots show the radical difference before and after reversing. The predictions have not been
   
 post processed yet in the plots but perform significantly better than before we reversed them.
  5. Methods we tried that did not work
      Method
Problem
Possible solutions
Wiener model
Frequencies were averaged
Use of direct feature set into different methods
No “look back” nature in feature set
Use R matrix instead
Feed forward feature selection
Greedy search
Principal Component Analysis
Linear vs. Ridge regression
Gave the same results
Choose a better regularizer (alpha)
Principal Component Regression
Lower correlation since some good features got left out
      
      Autoencoder for feature selection
Long train time over 400 epochs for each subject. Training over small number of epochs (20) gave equivalent results as PCR.
Train for longer
1D Convolutional Neural Network
Correlation at par with simple neural network. Probably insufficient data and eventual over fitting.
More data can be generated by increasing the number of windows (decreasing the window length and displacement while getting features)
  6. Thoughts about why the ring finger’s flexion was generally so correlated with
the middle and the little fingers
To deal with this specific observation, some anatomy was referred to. As we know, every finger is controlled by muscles to aid with movement. Some of these muscles are independent for each finger, for example some of the muscles in the little finger and most muscles in the thumb. However, the muscles that control the middle finger and ring finger are all common fingers. Therefore, it is hard to move the ring finger without the middle finger feeling some tug as well. Moreover, a single nerve branch (ulnar nerves) controls one half of the middle finger, whole ring finger and whole little finger, resulting in some form of interdependence between the three. Lastly, the nerves for the ring finger and little finger are intertwined. It is the same for the ring finger and middle finger but since the middle finger is affected by another nerve branch (radial nerve), it enjoys more independence. Ultimately, the ring finger gets the short end of the stick and this is why it’s almost really hard to move it without the middle finger and little finger moving as well.
7. Conclusion about your overall experience with the project
Overall the project led us to mainly think about feature selection and feature extraction from time series data. Although the homeworks introduced us to calculate basic features like line length and area, we were soon required to find features from the frequency domain. The moving windows method from the homeworks to get features benefited us in this project too. Reversing the order of the feature set proved to be the key bug in our code, which we fortunately evaded. Once this bug was resolved, the open ended nature of the competition allowed us to try a plethora of methods. We initially started with a forward feature selection method, which proved to be a greedy and naive way to approach the question : “Which
   
features are the best?”. We soon moved on to Linear Regression, Ridge Regression and Neural Networks. We used a fairly general neural network model by keeping its architecture common across all fingers and all subjects. We could have tried out an ensemble of these different architectures instead to make a more robust model. In our opinion, post processing the predictions improved the performance of the whole algorithm by a large amount. The whole project proved to be a great learning curve for each one of us.
8. References
1. Decoding Flexion of Individual Fingers using Electrocorticographic Signals in Humans by Kubanek et al. (2009).
2. Decoding Visual Information From a Population of Retinal Ganglion Cells by Warland et al. (1997).
3. Decoding Finger Flexion from Band-Specific ECoG Signals in Humans by Nanying Liang and Laurent Bougrain (2012).
