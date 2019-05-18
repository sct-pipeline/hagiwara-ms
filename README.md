# hagiwara-ms

The analysis pipeline available in this repository enables to output the following metrics (organized per contrast):

- **T2**: Spinal cord CSA averaged between C2 and C3.
- **T2s**: Gray matter CSA averaged between C3 and C4.
- **DWI**: FA in WM averaged between C2 and C5.
- **MTS**: MTR in WM averaged between C2 and C5. Uses MTon_MTS and MToff_MTS.
- **MTS**: MTSat & T1 map in WM averaged between C2 and C5. Uses MTon_MTS, MToff_MTS and T1w_MTS.

### Dependencies

MANDATORY:
- To convert DCM to BIDS structure: [dcm2bids](https://github.com/cbedetti/Dcm2Bids#install).
- This pipeline has beed tested using [SCT 4.0.0-beta.4](https://github.com/neuropoly/spinalcordtoolbox/releases/tag/4.0.0-beta.4).

OPTIONAL:
- For correcting segmentations, you can use [FSLeyes](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FSLeyes).
- For processing multiple subjects in parallel, you can use [GNU parallel](https://www.gnu.org/software/parallel/)

### Dicom to NIFTI conversion and data organization

To facilitate the collection, sharing and processing of data, we use the [BIDS standard](http://bids.neuroimaging.io/). An example of the data structure for one center is shown below:

~~~
spineGeneric_multiSubjects
├── dataset_description.json
├── participants.json
├── participants.tsv
├── sub-ucl01
├── sub-ucl02
├── sub-ucl03
├── sub-ucl04
├── sub-ucl05
└── sub-ucl06
    ├── anat
    │   ├── sub-ucl06_T1w.json
    │   ├── sub-ucl06_T1w.nii.gz
    │   ├── sub-ucl06_T2star.json
    │   ├── sub-ucl06_T2star.nii.gz
    │   ├── sub-ucl06_T2w.json
    │   ├── sub-ucl06_T2w.nii.gz
    │   ├── sub-ucl06_acq-MToff_MTS.json
    │   ├── sub-ucl06_acq-MToff_MTS.nii.gz
    │   ├── sub-ucl06_acq-MTon_MTS.json
    │   ├── sub-ucl06_acq-MTon_MTS.nii.gz
    │   ├── sub-ucl06_acq-T1w_MTS.json
    │   └── sub-ucl06_acq-T1w_MTS.nii.gz
    └── dwi
        ├── sub-ucl06_dwi.bval
        ├── sub-ucl06_dwi.bvec
        ├── sub-ucl06_dwi.json
        └── sub-ucl06_dwi.nii.gz
~~~

To convert your DICOM data folder to the compatible BIDS structure, we ask you
to install [dcm2bids](https://github.com/cbedetti/Dcm2Bids#install). Once installed,
convert your Dicom folder using the following
command (replace xx with your subject number):
~~~
dcm2bids -d <PATH_DICOM> -p sub-xx -c config_spine.txt -o CENTER_spineGeneric
~~~

For example:
~~~
dcm2bids -d /Users/julien/Desktop/DICOM_subj3 -p sub-C01 -c ~/Desktop/config_spine.txt -o my_subjects
~~~

A log file is generated under `tmp_dcm2bids/log/`.


### How to run

Copy and rename the parameter file:
~~~
cp parameters_template.sh parameters.sh
~~~

Edit the parameter file and modify the variables according to your needs:
~~~
edit parameters.sh
~~~

Launch processing
~~~
./run_process.sh process_data.sh
~~~

### Quality Control (Rapid)

A first quality control consists in opening the .csv results under `results/` folder
and spot values that are abnormality different than the group average.

Identify the site/subject/contrast associated with the abnormal value, and look at the
segmentation (or data). If the segmentation is clearly wrong, fix it (see [Quality Control (Slow)](#quality-control-slow). If the data look ugly (lots of artifact, motion, etc.), report it under a new file: `qc_report/$site_$subject_$contrast.txt`

### Quality Control (Slow)

After the processing is run, check your Quality Control (QC) report, by opening
double clicking on the file `qc/index.html`. Use the "Search" feature of the QC
report to quickly jump to segmentations or labeling results.

#### Segmentation

If you spot issues (missing pixels, leaking), identify the segmentation file, open
it with an editor (e.g., [FSLeyes](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FSLeyes)),
modify it (Tools > Edit Mode) and save it (Overlay > Save > Save to new file) with suffix `-manual`. Example: `sub-01_T2w_RPI_r_seg-manual.nii.gz`. Then, move the file to the folder you defined
under the variable `PATH_SEGMANUAL` in the file `parameters.sh`. Important: the manual segmentation
should be copied under a subfolder named after the site, e.g. `seg_manual/spineGeneric_unf/sub-01_T2w_RPI_r_seg-manual.nii.gz`. The files to look for are:

| Segmentation  | Associated image  | Relevant levels | Used for |
|:---|:---|:---|---|
| sub-XX_T1w_RPI_r_seg.nii.gz | sub-XX_T1w_RPI_r.nii.gz | C2-C3 | CSA |
| sub-XX_T2w_RPI_r_seg.nii.gz | sub-XX_T2w_RPI_r.nii.gz | C2-C3 | CSA |
| sub-XX_T2star_rms_gmseg.nii.gz | sub-XX_T2star_rms.nii.gz | C3-C4 | CSA |
| sub-XX_acq-T1w_MTS_seg.nii.gz | sub-XX_acq-T1w_MTS.nii.gz | C2-C5 | Template registration |

**Note:** For the interest of time, you don't need to fix *all* slices of the segmentation but only the ones listed
in the "Relevant levels" column of the table above.

#### Vertebral labeling

If you spot issues (wrong labeling), manually create labels in the cord at C2 and C5 mid-vertebral levels using the following command (you need to be in the appropriate folder before running the command):
~~~
sct_label_utils -i IMAGE -create-viewer 3,5 -o IMAGE_labels-manual.nii.gz
~~~
Example:
~~~
sct_label_utils -i sub-01_T1w.nii.gz -create-viewer 3,5 -o sub-01_T1w_labels-manual.nii.gz
mkdir ${PATH_SEGMANUAL}/spineGeneric_unf/
mv sub-01_T1w_labels-manual.nii.gz ${PATH_SEGMANUAL}/spineGeneric_unf/
~~~
Then, move the file to the folder you defined
under the variable `PATH_SEGMANUAL` in the file `parameters.sh`, as done for the segmentation.

Once you've corrected all the necessary files, re-run the whole process. Now, when the manual file exists,
the script will use it in the processing:
~~~
./run_process.sh process_data.sh
~~~

## Contributors

[List of contributors for the analysis pipeline.](https://github.com/sct-pipeline/spine_generic/graphs/contributors)

## License

The MIT License (MIT)

Copyright (c) 2018 Polytechnique Montreal, Université de Montréal

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
