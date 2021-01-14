# MSc thesis project "Super-resolution neural networks: a strategy to downscale model output data"
Running complex geophysical models takes a long computation time. Examples of this are the general circulation models, that are used to study the global climate system. These models run at low horizontal resolution (150-400km), because of their computational demand. Hydrological models require high-resolution meteorological data, and often rely on downscaling techniques to obtain high-resolution data from the low-resolution model output data of the general circulation models. In this research, we test a new statistical downscaling method: downscaling using deep neural networks. This method overcomes many of the problems that the currently used downscaling techniques face, and therefore has great potential in downscaling model output data. We built, trained and tested a deep convolutional auto-encoder to see the potential of this technique to downscale model output data, by downscaling meteorological (reanalysis) data as a case study. 

Below is a description of all files and scripts used for the MSc thesis research of Avelon Gerritsma:
Super-resolution neural networks: a strategy to downscale model output data

This document indicates the order in which the files were created/used, files are indicated with /. 
In the files are desciptions to explain what is done.
The re-analysis data are downloaded from https://www.ecmwf.int/en/forecasts/dataset/ecmwf-reanalysis-20th-century-using-surface-observations-only
Intermediate files (TFRecord files that are rescaled, after step 3) can be found on 4TU.nl name: Data underlying the research on: Downscaling precipitation

The first step was to download the ERA20C data, calculate total daily precipitation sum and upload data to google earth engine (GEE). Use file:
/1.Download_ERA20C.ipynb 

In GEE, image tiles are created and uploaded to the bucket as TFRecord files:
/2.Scripts_GEE.txt
Four scripts are included in this file: first is to create image tiles and export them to the bucket as TFRecord files, 
second is to export lon-lat data (coordinate data of TFrecord files with the precipitation tiles), third is to plot data to look
at precipitation map in GEE and lastly a script to download tiles as geotiff (with coordinate data).

Next step is to open the image tiles in Jypyterlab/Colab and normalize the data, this is done in two steps:
1. data^0.2
2. MinMaxScaler fitted on high resolution data
File:
/3.Normalize_data.ipynb 
'scaler_new.pkl' is the fitted scaler, stored in data folder.
After the scaler is defined, this scaler is used to rescale all the data. The image tiles are cut into patches and saved 
in big TFRecord files (quicker to reload all the data for training Neural network).

In \4a.Deep_convolutional_auto-encoder.ipynb  is the auto-encoder programmed with tensorflow. First data is loaded 
(big files created with former step). Model is set-up, compiled. Custom loss function is defined. At model.compile, 
you can choose between loss=MSE or loss=loss_function == custom loss.
Then model.summary and model.plot show a schematic representation of the neural network.
Model is trained for the nr of epochs that is specified in the script (e.g.epochs=10)

In \5.Predition.ipynb the high resolution patches are predicted by the neural network for the validation years. 
These are the patches that are on a different location compared to the training patches.
First model is loaded, and compiled in case you defined a custom loss function.
After all patches are downscaled, a scaling factor is defined because of the volume precipitation loss (see section scaling in discussion).
All patches are multiplied with a scaling factor and are then saved in one netcdf file, with metadata.
This netcdf file is then uploaded to the bucket.

The figures showing the downscaled patches are made with \6.Maps_report.ipynb 
You can choose day to make the plots for.
At the end of the document, the MS-SSIM is defined, used in discussion report.

Patches that are on the same location as the training patches, are saved in big TFRecord files in:
/7.Patch_EU_train, and downscaled in /8.Predict_patch_train
In this file, high resolution patches are predicted by the neural network for the validation years, 
for patches that are on the same location as one of the training patches. It also contains code to plot
the results. Since the TFRecord files contain no coordinate information, this information is retrieved 
by exporting a geotiff map from GEE with lon-lat bands. By cutting out the same patch as for the 
precipitation data, coordinate data is retrieved and added to the data array.  
After all patches are downscaled, a scaling factor is defined because of the volume precipitation loss (see section scaling in discussion).
All patches are multiplied with a scaling factor and are then saved in one netcdf file with coordinate data and metadata.
