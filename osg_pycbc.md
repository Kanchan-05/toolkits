## How to submit PyCBC workflow on OSG 

### Get a conda enviroment 

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh 
rm Miniconda3-latest-Linux-x86_64.sh
```

Check the version of singularity used for PyCBC via
```
/cvmfs/singularity.opensciencegrid.org/pycbc/ 
```
Download the PyCBC version with tag that exists in the above list

```
git clone https://github.com/gwastro/pycbc.git
git check [tag_version]
```

### Generate a image.def 

To generate a `image.def` from scratch, follow the instructions given at [LINK](https://portal.osg-htc.org/documentation/htc_workloads/using_software/containers-singularity/)

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
    conda create -y -n "subsolar" python=3.11
    conda activate myenv
    # Install pycbc and other dependencies
    git clone https://github.com/gwastro/pycbc.git
    cd pycbc
    pip install .
    pip install numpy==2.1.3
    pip install -r requirements.txt
    pip install mkl
    conda update -n base -c defaults conda
    conda install  -y -n "myenv" conda-forge::pyfftw
    conda install  -y -n "myenv" conda-forge::fftw
    conda install  -y -n "myenv"  -c conda-forge gitpython
    conda install  -y -n "myenv"  -c conda-forge gsl pygsl_lite
    
    # Install Pegasus GPG key
    curl https://download.pegasus.isi.edu/pegasus/gpg.txt | apt-key add -
    # Add Pegasus repository to sources list
    echo 'deb https://download.pegasus.isi.edu/pegasus/ubuntu focal main' > /etc/apt/sources.list.d/pegasus.list
%environment
    # set up environment for when using the container
    . /opt/conda/etc/profile.d/conda.sh
    conda activate myenv
     
```

### Generate a signularity container 
```
apptainer build pycbc_container.sif image.def 
```

  
