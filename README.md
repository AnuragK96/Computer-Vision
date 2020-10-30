## Attempt for pre-processing pipeline of data

Raw BOLD image of two co-registered runs, and the signal from the same occipital lobe voxel was not successfully completed. 

### Packages Used
To implement the pipeline, the following python packages are used
 * ```os``` 
 * ```numpy``` 
 * ```skimage```
 * ```pandas```
 * ```nibabel```

### Pipeline implemented 

The steps taken implement the pipeline are as follows:

1) Collect data from the https://openneuro.org/datasets/ds001506/versions/1.3.1 website along with the tsv and nii files.
2) Take data from tsv file for getting information about indexing and corresponding image files used
3) The fMRI data was spatially and temporally normalised and the 3D fMRI data and corresponding image data are obtained. 

### Code

```
import os 
import numpy as np
from skimage.io import imread
import pandas as pd
import nibabel as nib

def file_read(tsv_file, fmri_file, img_path):
    image = nib.load(fmri_file)
    
    df = DataFrame.from_csv(tsv_filename, sep="\t")
    df['onset'] = df.index
    time = df['onset'].divide(2)
    df = df[1:-1]
    
    
    j = 0
    avg_images = []
    stim_images = []
    norm_images_temp = np.zeros([images.shape[0], images.shape[1],images.shape[2]])
    for indices in time:
        indices = int(indices)
        imgages = images.dataobj[:,:,:,indices+2:indices+6]
        #average image out temporally
        avg = np.average(images,axis = -1)
        #normalise image spatially
        for i in range(avg.shape[2]):
            norm_images_temp[:,:,i] = (avg[:,:,i] - np.mean(avg[:,:,i]))/np.std(avg[:,:,i])
        avg_images.append(norm_images_temp)
        stim_images.append(imread(img_path+df['stimulus_name'].iloc[j]))
        j+=1

    return (avg_images, stim_images)


cwd = os.getcwd()
fmri_path = os.path.join(cwd,"/fmri_files/")
fmri_files = os.listdir(fmri_path)

tsv_path = os.path.join(cwd,"/tsv_files/")
tsv_files = os.listdir(tsv_path)

image_path = cwd+"/images/"


fmri = []
image = []

for i in range(len(fmri_files)):
    tsv_file = tsv_files[i]
    fmri_file = fmri_files[i]
    fun_fmri, fun_img = file_read(tsv_files, fmri_files)
    fmri.append(fun_fmri)
    image.append(fun_img)
```
The bottleneck was at the conversion of 3D fmri files to 2D images and thereafter how to correlate between the slices and the stimulating image.

To overcome this, we used a pre-trained VGG19 model which was used as a decoder for the f-MRI scans. (https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1006633). 
Using this pre-trained model, we were able to generate the required image using our algorithm developed on top of this. 