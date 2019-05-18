# hagiwara-ms

Pipeline for processing MS patients

# spine-generic

Description of the publicly-available database and processing pipeline for the "spine generic protocol" project.

- [Data collection and organization](#data-collection-and-organization)
- [Analysis pipeline](#analysis-pipeline)

## Data collection and organization

The "Spine Generic" MRI acquisition protocol is available at [this link](https://osf.io/tt4z9/). Each site scanned six healthy subjects (3 men, 3 women), aged between 20 and 40 y.o. Note: there is a flexibility here, and if you wish to scan more than 6 subjects, you are welcome to. If your site is interested in participating in this publicly-available database, please contact Julien Cohen-Adad for details.

### Data conversion: DICOM to BIDS

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
[download this config file](https://raw.githubusercontent.com/sct-pipeline/spine-generic/master/config_spine.txt) (click File>Save to save the file), then convert your Dicom folder using the following
command (replace xx with your center and subject number):
~~~
dcm2bids -d <PATH_DICOM> -p sub-xx -c config_spine.txt -o CENTER_spineGeneric
~~~

For example:
~~~
dcm2bids -d /Users/julien/Desktop/DICOM_subj3 -p sub-milan03 -c ~/Desktop/config_spine.txt -o milan_spineGeneric
~~~

A log file is generated under `tmp_dcm2bids/log/`. If you encounter any problem while
running the script, please [open an issue](https://github.com/sct-pipeline/spine-generic/issues)
and upload the log file. We will offer support.

Once you've converted all subjects for the study, create the following files and add them to the data structure:
