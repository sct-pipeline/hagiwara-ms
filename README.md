# hagiwara-ms

Pipeline for processing MS patients

# spine-generic

Description of the publicly-available database and processing pipeline for the "spine generic protocol" project.

- [Data collection and organization](#data-collection-and-organization)
- [Analysis pipeline](#analysis-pipeline)

## Data collection and organization

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
