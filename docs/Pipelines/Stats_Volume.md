# `statistics-volume` - Volume-based mass-univariate analysis with SPM

This pipeline performs statistical analysis (currently group comparison) on volume-based features using the general linear model (GLM) [[Friston et al., 1994](https://doi.org/10.1002/hbm.460020402)]. To that aim, the pipeline relies on the tools available in [SPM](http://www.fil.ion.ucl.ac.uk/spm/).

Volume-based measurements are analyzed in the [IXI549Space](https://bids-specification.readthedocs.io/en/stable/99-appendices/08-coordinate-systems.html#standard-template-identifiers) (from SPM12). Currently, this pipeline mainly handles gray matter maps obtained from T1 images using the [`t1-volume` pipeline](../T1_Volume) and standardized uptake value ratio (SUVR) maps obtained from FDG PET data using the [`pet-volume` pipeline](../PET_Volume).

## Dependencies
<!--If you installed the docker image of Clinica, nothing is required.-->

If you only installed the core of Clinica, this pipeline needs the installation of **Matlab** and **SPM**, or of **SPM standalone**, on your computer. You can find how to install these software packages on the [third-party](../../Third-party) page.

## Running the pipeline
The pipeline is divided into two sub-pipelines:

- `statistics-volume`: performs group comparison but no statistical correction. It generates an SPM report necessary to perform statistical corrections using the second sub-pipeline.
- `statistics-volume-correction`: performs family-wise error rate (FWE) or false discovery rate (FDR ) correction at the peak/cluster level. The user has to report the values from the SPM report generated by `statistics-volume` in order to run this pipeline.

### `statistics-volume` pipeline
The pipeline can be run with the following command line:

```
clinica run statistics-volume caps_directory subject_visits_with_covariates_tsv contrast feature_type group_id
```
where:

  - `caps_directory` is the output folder containing  the results of the [`t1-volume`](../T1_Volume) or [`pet-volume`](../PET_Volume) pipeline and the output of the present command, both in a [CAPS hierarchy](../../CAPS/Introduction).
  - `subject_visits_with_covariates_tsv` is a TSV file containing a list of subjects with their sessions and all the covariates and factors of the model (the content of the file is explained in the [Example](../Stats_Surface/#comparison-analysis) subsection of the `statistics-surface` pipeline).
  - `contrast` is a string defining the contrast matrix or the variable of interest for the GLM, e.g. `group` or `age`.
  - `feature_type` indicates to Clinica which volume-based data to use. It can be either `graymatter` (outputs of [`t1-volume`](../T1_Volume)) or `fdg` (outputs of [`pet-volume`](../PET_Volume)). Use `'custom'` if you want to use the `--custom_files` flag (more [below](#advanced-specifying-what-volume-data-to-use)).
  - `group_id` defines the group name for the analysis.

Optional parameters:

  - `--group_id_caps` is used when you have multiple groups in your CAPS and Clinica is not able to determine which one to choose when reading inputs.
  - `-fwhm` is the full width at half maximum (FWHM) of the smoothing used in your input file (by default 8 (mm), i.e. the default value of the [`t1-volume`](../T1_Volume)) and [`pet-volume`](../PET_Volume) pipelines)).

### `statistics-volume-correction` pipeline
Once the `statistics-volume` sub-pipeline has finished, you need to open the SPM report (`report1.png` or `report2.png` file). This will look like as follows:

![SPM report](https://user-images.githubusercontent.com/49677712/75558316-f0f23280-5a41-11ea-9489-be40ee66ec16.png)

You will need to report the following information in the `statistics-volume-correction` pipeline:

  - `height_threshold`: T value corresponding to an uncorrected p-value of 0.001
  - `FWEp`: height threshold (i.e. voxel-level (= peak) threshold)
  - `FDRp`: height threshold (i.e. voxel-level (= peak) threshold)
  - `FWEc`: extent threshold (i.e. cluster size threshold)
  - `FDRc`: extent threshold (i.e. cluster size threshold)

The pipeline can then be run with the following command line:

```
clinica run statistics-volume-correction caps_directory t_map height_threshold FWEp FDRp FWEc FDRc
```
where:
  - `t_map`: name of the T statistic map used for the correction

Optional parameters:

  - `n_cuts`: number of cuts to display in the final figure


## Outputs

Results are stored in the following folder of the [CAPS hierarchy](../../CAPS/Specifications/#statistics-volume-volume-based-mass-univariate-analysis-with-spm): `groups/group-<group_label>/statistics_volume/group_comparison_measure-<measure-label>`. The most important files are:

  - `group-<group_id>_participants.tsv`: copy of the `subject_visits_with_covariates_tsv` parameter file.
  - `group-<grp>_<grp_1>-lt-<grp_2>_measure-<msr>_fwhm-<fwhm>_TStatistics.nii`: T statistics associated with the hypothesis group1 < group2.
  - `group-<grp>_mask.nii`: voxels included in the analysis.
  - `group-<grp>_report.png`: all the results of the 2 sample t-test generated by SPM. Contains information necessary to use the `statistics-volume-correction` sub-pipeline.

The `<group_1>-lt-<group_2>` means that the tested hypothesis is: "the measurement of `<group_1>` is lower than (`lt`) the measurement of `<group_2>`". The pipeline includes both contrasts so `*<group_2>-lt-<group_1>*` files are also saved.


The full list of output files from the `statistics-volume-[correction]` pipeline can be found in the
[The ClinicA Processed Structure (CAPS) specifications](../../CAPS/Specifications/#statistics-volume-volume-based-mass-univariate-analysis-with-spm).


## Describing this pipeline in your paper
!!! cite "Example of paragraph:"
    These results have been obtained using the `statistics-volume` pipeline of Clinica [[Routier et al](https://hal.inria.fr/hal-02308126/)]. This pipeline is a wrapper of the statistical analysis toolbox implemented in [SPM](http://www.fil.ion.ucl.ac.uk/spm/). More precisely, a point-wise, voxel-to-voxel model was used to conduct a group comparison of whole brain voxels. The data were smoothed using a Gaussian kernel with a full width at half maximum (FWHM) set to `<FWHM>` mm. The general linear model was used to control for the effect of `<covariate_1>`, ... and  `<covariate_N>`.

    - For FWEp: Statistics were corrected for multiple comparisons using the family-wise error (FWE) correction at the peak level with a statistical threshold of P < 0.05 FWE.
    - For FWEc: Statistics were corrected for multiple comparisons using the family-wise error (FWE) correction at the cluster level. A statistical threshold of P < `<ClusterThreshold>` was first applied (height threshold). An extent threshold of P < 0.05 corrected for multiple comparisons was then applied at the cluster level.
    - For FDRc: Statistics were corrected for multiple comparisons using the false discovery rate (FDR) correction at the cluster level. A statistical threshold of P < `<ClusterThreshold>` was first applied (height threshold). An extent threshold of P < 0.05 corrected for multiple comparisons was then applied at the cluster level.

!!! tip
    Easily access the papers cited on this page on [Zotero](https://www.zotero.org/groups/2240070/clinica_aramislab/collections/ACBHQWPB).

## Support
-   You can use the [Clinica Google Group](https://groups.google.com/forum/#!forum/clinica-user) to ask for help!
-   Report an issue on [GitHub](https://github.com/aramis-lab/clinica/issues).


## (Advanced) Using volumes other than gray matter or FDG PET SUVR maps

If you run the help command line `clinica run statistics-volume -h`, you will find the following flag:

 - `--custom_files CUSTOM_FILES`: it allows you to specify which file should be taken in the `CAPS/subjects` directory. For example, if you want to use the file `sub-<subject_label>_ses-<session_label>_task-rest_acq-av45_pet_space-Ixi549Space_pet.nii.gz` that is contained in `CAPS/subjects/sub-<subject_label>/ses-<session_id>/pet/preprocessing/group-<group_label>`, you can use the argument `*sub-*_ses-*_task-rest_acq-av45_pet_space-Ixi549Space_pet.nii.gz`. If you want to specify the group, you can use `group-<group_label>/sub-*_ses-*_task-rest_acq-av45_pet_space-Ixi549Space_pet.nii.gz`


This flag is read by Clinica when `feature_type` is neither `graymatter` nor `fdg`. The value you set in `feature_type` will appear in the `_measure-<feature_type>` key/value of the output files once the pipeline has run.