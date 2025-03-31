## How to submit PyCBC workflow on OSG 

### Get a conda enviroment 

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh 
rm Miniconda3-latest-Linux-x86_64.sh
```

If you are using the pre-defined singularity images then first check the version of singularity used for PyCBC via
```
/cvmfs/singularity.opensciencegrid.org/pycbc/ 
```
and download the PyCBC version with tag that exists in the above list

```
git clone https://github.com/gwastro/pycbc.git
git check [tag_version]
```

### Generate image.def 

If you want to create your own singularity container then first generate `image.def` from scratch, follow the instructions given at [LINK](https://portal.osg-htc.org/documentation/htc_workloads/using_software/containers-singularity/)

For example:
```
Bootstrap: docker
From: opensciencegrid/osgvo-ubuntu-20.04:latest
%post
    export DEBIAN_FRONTEND=noninteractive
    # Update package lists
    apt-get update -y
    # Install necessary packages
    apt-get install -y software-properties-common
    apt-get install -y \
            build-essential \
            cmake \
            g++ \
            r-base-dev \
            wget \
            Gnupg

    # install miniconda env
    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
    bash Miniconda3-latest-Linux-x86_64.sh -b -f -p /opt/conda
    rm Miniconda3-latest-Linux-x86_64.sh
    # install conda components - add the packages you need here
    . /opt/conda/etc/profile.d/conda.sh
    conda create -y -n "myenv" python=3.11
    conda activate myenv

    # Install pycbc and other dependencies
    git clone https://github.com/gwastro/pycbc.git
    cd pycbc
    pip install .
    pip install -r requirements.txt
    pip install mkl
    conda update -n base -c defaults conda
    conda install  -y -n "myenv" conda-forge::pyfftw
    conda install  -y -n "myenv" conda-forge::fftw
    conda install  -y -n "myenv"  -c conda-forge gitpython
    conda install  -y -n "myenv"  -c conda-forge gsl pygsl_lite
    
%environment
    # set up environment for when using the container
    . /opt/conda/etc/profile.d/conda.sh
    conda activate myenv
     
```
The above example will allow you to generate a conda environment inside the signularity container. This is useful because it gives you freedom to install any user-defined packages for workflow development purposes.

### Generate a signularity container 
```
apptainer build pycbc_container.sif image.def 
```

### Modify executable.ini for OSG 

```[executables]
average_psd = singularity:///usr/local/bin/pycbc_average_psd
bank2hdf = singularity:///usr/local/bin/pycbc_coinc_bank2hdf
calculate_psd = singularity:///usr/local/bin/pycbc_calculate_psd
coinc = singularity:///usr/local/bin/pycbc_coinc_findtrigs
combine_statmap = singularity:///usr/local/bin/pycbc_combine_statmap
fit_by_template = singularity:///usr/local/bin/pycbc_fit_sngls_by_template
fit_over_param = singularity:///usr/local/bin/pycbc_fit_sngls_over_multiparam
foreground_censor = singularity:///usr/local/bin/pycbc_foreground_censor
;hdfinjfind =singularity:///usr/local/bin/ycbc_coinc_hdfinjfind
hdf_trigger_merge = singularity:///usr/local/bin/pycbc_coinc_mergetrigs
;inj2hdf = singularity:///usr/local/bin/pycbc_convertinjfiletohdf
;inj_cut = singularity:///usr/local/bin/pycbc_inj_cut
;injections = singularity:///usr/local/bin/lalapps_inspinj
inspiral = singularity:///usr/local/bin/pycbc_inspiral

merge_psds =singularity:///usr/local/bin/pycbc_merge_psds
;optimal_snr = singularity:///usr/local/bin/pycbc_optimal_snr
;optimal_snr_merge = singularity:///usr/local/bin/pycbc_merge_inj_hdf
page_foreground = singularity:///usr/local/bin/pycbc_page_foreground
page_ifar = singularity:///usr/local/bin/pycbc_page_ifar
page_ifar_catalog = singularity:///usr/local/bin/pycbc_ifar_catalog
;page_injections =singularity:///usr/local/bin/pycbc_page_injtable
page_segplot = singularity:///usr/local/bin/pycbc_page_segplot
page_segtable = singularity:///usr/local/bin/pycbc_page_segtable
page_versioning = singularity:///usr/local/bin/pycbc_page_versioning
page_vetotable = singularity:///usr/local/bin/pycbc_page_vetotable
plot_bank = singularity:///usr/local/bin/pycbc_plot_bank_bins
plot_binnedhist = singularity:///usr/local/bin/pycbc_fit_sngls_split_binned
plot_coinc_snrchi = singularity:///usr/local/bin/pycbc_page_coinc_snrchi
plot_foundmissed = singularity:///usr/local/bin/pycbc_page_foundmissed
plot_gating = singularity:///usr/local/bin/pycbc_plot_gating
plot_hist = singularity:///usr/local/bin/pycbc_plot_hist
plot_qscan = singularity:///usr/local/bin/pycbc_plot_qscan
plot_range = singularity:///usr/local/bin/pycbc_plot_range
plot_segments = singularity:///usr/local/bin/pycbc_page_segments
plot_sensitivity = singularity:///usr/local/bin/pycbc_page_sensitivity
plot_singles = singularity:///usr/local/bin/pycbc_plot_singles_vs_params
plot_snrchi = singularity:///usr/local/bin/pycbc_page_snrchi
plot_snrifar = singularity:///usr/local/bin/pycbc_page_snrifar
plot_spectrum = singularity:///usr/local/bin/pycbc_plot_psd_file
plot_throughput = singularity:///usr/local/bin/pycbc_plot_throughput
results_page = singularity:///usr/local/bin/pycbc_make_html_page
splitbank = singularity:///usr/local/bin/pycbc_hdf5_splitbank
statmap = singularity:///usr/local/bin/pycbc_coinc_statmap
;statmap_inj = singularity:///usr/local/bin/pycbc_coinc_statmap_inj

sngls = singularity:///usr/local/bin/pycbc_sngls_findtrigs
;finds candidate triggers for single detectors
sngls_statmap = singularity:///usr/local/bin/pycbc_sngls_statmap
;The program combines output files generated by pycbc_sngls_findtrigs to generate a mapping between SNR and FAP/FAR, along with producing the combined foreground and background triggers.
;sngls_statmap_inj = singularity:///usr/local/bin/pycbc_sngls_statmap_inj
exclude_zerolag = singularity:///usr/local/bin/pycbc_exclude_zerolag


; #################### Mini Followup #########################################
foreground_minifollowup = singularity:///usr/local/bin/pycbc_foreground_minifollowup
;injection_minifollowup = singularity:///usr/local/bin/pycbc_injection_minifollowup
singles_minifollowup = singularity:///usr/local/bin/pycbc_sngl_minifollowup

html_snippet = singularity:///usr/local/bin/pycbc_create_html_snippet
;page_injinfo = singularity:///usr/local/bin/pycbc_page_injinfo
page_coincinfo = singularity:///usr/local/bin/pycbc_page_coincinfo
page_snglinfo = singularity:///usr/local/bin/pycbc_page_snglinfo
plot_trigger_timeseries = singularity:///usr/local/bin/pycbc_plot_trigger_timeseries
single_template_plot = singularity:///usr/local/bin/pycbc_single_template_plot
single_template = singularity:///usr/local/bin/pycbc_single_template
plot_singles_timefreq = singularity:///usr/local/bin/pycbc_plot_singles_timefreq
plot_snrratehist = singularity:///usr/local/bin/pycbc_page_snrratehist
plot_waveform = singularity:///usr/local/bin/pycbc_plot_waveform

[only-do]
inspiral = True

[common]
image_path = "osdf:///ospool/ap21/data/ksoni01/pycbc_container.sif"

[pegasus_profile]
condor|request_memory = 10000
condor|request_disk = 1000000
; logs are supposed to be in $HOME
pycbc|submit-directory = /home/ksoni01/work/proj_subsolar/pycbc_logs

[pegasus_profile-osg]
condor|My.SingularityImage =  ${common|image_path}
condor|+OpenScienceGrid = true
condor|Requirements = HAS_SINGULARITY == TRUE

[pegasus_profile-inspiral]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
condor|+InitialRequestMemory = 3000
condor|request_memory = ifthenelse( (LastHoldReasonCode=!=34 && LastHoldReasonCode=!=26), InitialRequestMemory, int(2 * NumJobStarts * MemoryUsage) )
condor|requirements = (HAS_SINGULARITY =?= TRUE) && (HAS_LIGO_FRAMES =?= True) && (IS_GLIDEIN =?= True) && (HAS_CVMFS_gwosc_osgstorage_org == True)
condor|periodic_hold = (JobStatus == 2) && ((CurrentTime - EnteredCurrentStatus) > (2 * 86400))
condor|periodic_release = ((HoldResasonCode =?= 46) || (HoldReasonCode =?= 34) || (HoldReasonCode =?= 26)) || ((JobStatus == 5) && (HoldReasonCode == 3) && (NumJobStarts < 5) && ((CurrentTime - EnteredCurrentStatus) > (300)))
condor|periodic_remove = (NumJobStarts >= 5)
condor|request_disk = 2637154
dagman|retry = 10000
[pegasus_profile-average_psd]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-bank2hdf]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-calculate_psd]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
pycbc|site-scratch = /ospool/ap21/data/ksoni01/work/proj_subsolar/installations/example_search/output
condor|+InitialRequestMemory = 10000
condor|request_disk = 1000000
condor|request_cpus = ${calculate_psd|cores}
dagman|priority = 5000
dagman|retry = 10
[pegasus_profile-combine_statmap]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-exclude_zerolag]
pycbc|site = osg
condor|My.SingularityImage =${common|image_path}
[pegasus_profile-fit_by_template]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-fit_over_param]
pycbc|site = osg
condor|My.SingularityImage =${common|image_path}
[pegasus_profile-foreground_censor]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-hdfinjfind]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-hdf_trigger_merge]
pycbc|site = osg
condor|My.SingularityImage =${common|image_path}
condor|transfer.bypass.input.staging = true
[pegasus_profile-inj2hdf]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-inj_cut]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-injections]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-merge_psds]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-optimal_snr]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-optimal_snr_merge]
pycbc|site = osg
condor|+SingulariyImage = ${common|image_path}
[pegasus_profile-page_foreground]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_ifar]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_ifar_catalog]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_injections]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_segplot]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_segtable]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_versioning]
pycbc|site = osg
condor|My.SingularityImage =${common|image_path}
[pegasus_profile-page_vetotable]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_bank]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_binnedhist]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_coinc_snrchi]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_foundmissed]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_gating]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_hist]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_qscan]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_range]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_segments]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_sensitivity]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_singles]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_snrchi]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_snrifar]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_spectrum]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_throughput]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-results_page]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-sngls]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-sngls_statmap]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-sngls_statmap_inj]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-splitbank]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-statmap]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-statmap_inj]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-strip_injections]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-tmpltbank]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-html_snippet]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-foreground_minifollowup]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-injection_minifollowup]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-singles_minifollowup]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_injinfo]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_coincinfo]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-page_snglinfo]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_trigger_timeseries]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-single_template_plot]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-single_template]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_singles_timefreq]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
[pegasus_profile-plot_snrratehist]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path} 
[pegasus_profile-plot_waveform]
pycbc|site = osg
condor|My.SingularityImage = ${common|image_path}
```

