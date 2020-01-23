# Self Supervised Retinal Thickness prediction using Deep Learning

The repository contains the code and partial data for for the paper: https://www.biorxiv.org/content/10.1101/861757v1

The repository contains 5 main parts.

* Thickness segmentation
* Thickness map calculation
* Thickness prediction
* SSL Kaggle
* Retinal screening evaluation

## Getting Started

To set up a working example, clone the github repository and install all software requirements listed in the requirements.txt. Main tools used are Python and Tensorflow.

In order to run illustrative examples four downloads are required from: https://doi.org/10.5281/zenodo.3626020

Below are the four necessary file to download:

* DeepRT_light.tar.gz
* ssl_data_sample.tar.gz
* thickness_segmentation_data.tar.gz
* thickness_segmentation_model.tar.gz

1. Extract DeepRT_light.tar.gz and place the file in ~/ssl_kaggle/pretrained_weights directory.
2. Extract ssl_data_sample.tar.gz and put in the ~/ssl_kaggle as the data folder
3. Extract thickness_segmentation_data.tar.gz put in ~/thickness_segmentation
4. Extract thickness_segmentation_model and put in ~/thickness_map_calculation/output folder

Once repository is set up locally with all code and correctly placed data, follow below instructions for each step.

#### Thickness segmentation

In the folder ./thickness_segmentation/data please locate all images and segmentation masks in the ~/all_images and ~/all_labels folder.

In the ~/file_names_complete folder please place 3 .csv files containing the file id's for train, validation and test image named:

* test_new_old_mapping.csv
* train_new_old_mapping.csv
* validation_new_old_mapping.csv

These csv files should contain a column names "new_id" which contains correspond to the image ids present in the ~/all_images and ~/all_labels folders.

Note: The algorithm is trained on OCT images from both Spectralis and Topcon devices. Hence the different .csv mapping different id's in the ~/data directory. Here one can change the logic of reading in files by editing the main.py and python_generator.py python files.

To train the algorithm, run the main.py file with parameters set in the params.json config file (learning rate, epochs among others, see params.json).

The main.py file creates a model directory in the ~/logs directory. To evaluate the model, run evaluation.py with specified model directory directly in the python program. The evaluation.py program will mainly do two things.

* save plots of OCT, annotation and predicted annotation mask in the ~/model_dir/test_predictions folder. 
* save a result.csv file logging the IOU score for each induvidual test record, later to be used for plotting.

#### Thickness map calculation

To calculate high resolution thickness maps based on OCT DICOM exports store all DICOM files the ~/data directory. In the example directory in this repository the ~/data folder contains subfolders with the following names:

* PatientPseudoID_Laterality_StudyDate

Where Patient pseudo ID is a randomized ID given to a patient at the time of data export. Laterality referring to the Right or the Left eye and StudyDate being the date of the eye examination.

In the ~/output directory, place the trained weights for the OCT segmentation algorithm obtained in the thickness segmentation task.

Also create a ~/thickness_maps directory for saving the calculated thickness maps in numpy format.

To calculate high resolution thickness maps for the DICOM files in the ~/data directory, configure the following parameters in the main.py file:

* img shape = (256, 256, 3)
* save_octs = True
* save_segmentations = True
* thickness_map_dim = 128

Note: img_shape should be the same OCT dimension used in the thickness segmentation task. save_octs and save_segmentations is by default set to true, this saves each individual OCT and tissue segmentation in the DICOM directory. Thickness map dim is set to 128 as determined sufficient for accurate thickness map regression.

As stated in paper, the quality of the ground truth thickness map can vary considerably depending on various factors. Most importantly, low quality OCT scans present in real world data can cause low quality examples. For this reason, in ~/quality_filter total variation is calculated for each record and saved in ~/quality_filter/tv_pd.csv. This csv can then be used to filter out to low quality thickness maps from training and evaluation. The value of the choosen threshold will affect the reported results of the thickness regressor. In this project the value 0.014 was choosen after visual inspection of several maps and their quality, on both sides of this threshold.
 
To generate the tv_ps.csv run log_tv.py and ensure that the path to the calculated thickness map is set directly in the log_tv.py.

With the example DICOM files provided all have a total variation below the threshold of 0.014 which corresponds to 4.19 micro meters as stated in the paper. 

#### Thickness map prediction

In order to run the Thickness prediction model (DeepRT) place all your coregistered fundus images and thickness maps in ~/data/fundus and ~/data/thickness_maps as in data sample provide. Further in ~/data/filenames provide .csv files for train, validation and test samples as in test sample. 

Before training the model, configure params.json, here the default parameters are set as used in paper. "enc_filters" here regulates the size of the encoder. In the paper the results are achieved using "enc_filters" = 8 for the DeepRT light model. 

Once trained, find model in the ~/logs directory.

To evaluate the model and produce all the test statistics used for the plots in the manuscript, simply run evaluation.py with the params.model_directory set correctly in the python file. This program will generate all necessary test statistics in the results_log.pd file and save it in the specified model directory. Further, all labels and prediction are plotted and saved in the folder "predictions", also residing in the specified model directory.   

Note: in order to produce results on thickness prediction test errors for edema and atrophy cases, put the file gold_standard.csv in the ~/data/gold_standard directory, see test data for example format. Important here is that the file names are of the same format as in other examples.  

#### Transfer learning onto the Kaggle Diabetic Retinopathy data set

In order to run the evaluation of transferring DeepRT weights vs. random and Imagenet initialization the first step is to download the data publicly available at: https://www.kaggle.com/c/diabetic-retinopathy-detection/data.

Note: here only the public test set is used for evaluation the algorithms. 

The second step is to pre-process the data as stated in the manuscript. To preprocess the image run preprocess.py in the ~/data directory. The repository includes 5 example images located in ~/data/unprocessed. After running the preprocess.py the output is saved in ~/data/processed.

Once all the downloaded images are preprocessed the data partitions can be created. An example is seen in the ~/data/512_3 directory. Here the test folder always contains the same image accross all partitions. In this example the 3 % partition is included in the repository where each image is resized to 512x512x3 pixels. In this repository the test folder contains the same images as the validation data, to keep the amount of data in the repository low.

Once all data is structured, run main.py specifying the model, weight initialization as well has hyperparameters. 

Note: Using grid search the optimal parameters for ResNet50 with Imagenet Init was found to be learning rate 0.0001 and momentum 0.99. While for DeepRT random and transfer used learning rate 0.001 and momentum 0.9.

Model directories are as usual logged in the ~/logs folder. To evaluate the model on the test data, specify the model directory and model of interest  in the evaluation.py file and run it. A result.txt file with all the main metrics will be saved in the model directory among others. 

### Prerequisites

## Authors

* **Olle Holmberg**: https://www.linkedin.com/in/olle-holmberg-2ba23152/

## Acknowledgments

