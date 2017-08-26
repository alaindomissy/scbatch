BootStrap: docker
From:  ubuntu:16.04

  #############################################################################

%setup

  # initial setups from outside the container
  # this is run from outside the container to start setting it up

  echo "Looking in directory '$SINGULARITY_ROOTFS' for /bin/sh"
  if [ ! -x "$SINGULARITY_ROOTFS/bin/sh" ]; then
      echo "Hrmm, this container does not have /bin/sh installed..."
      exit 1
  fi

  mkdir -p $SINGULARITY_ROOTFS/oasis/tscc/scratch
  mkdir -p $SINGULARITY_ROOTFS/projects/ps-yeolab
  mkdir -p $SINGULARITY_ROOTFS/projects/ps-yeolab3
  mkdir -p $SINGULARITY_ROOTFS/projects/ps-scrm
  mkdir -p $SINGULARITY_ROOTFS/oasis/projects/nsf

  mkdir -p $SINGULARITY_ROOTFS/media/mis

  mkdir -p $SINGULARITY_ROOTFS/opt/members
  mkdir -p $SINGULARITY_ROOTFS/opt/patches

  # these are needed early for post section
  cp -r ./members/* $SINGULARITY_ROOTFS/opt/members/
  cp -r ./patches/* $SINGULARITY_ROOTFS/opt/patches/

  #############################################################################

%post

  # running post scriptlet
  # this is run inside the container to install all necessary packages

  set -o xtrace
  set -o nounset
  set -o errexit
  #set -o pipefail

  # ubuntu does not have bash in /usr/bin/env ??
  ln -s /bin/bash /usr/bin/bash

  ########
  # UBUNTU
  apt-get -y update
  apt-get -y install make gcc g++ zlib1g-dev libncurses5-dev nano unzip
  # cleanup   x? M
  apt-get clean
  g++ --version

  # fix for /bin/gtar: not found when running devtools::install_git()
  ln -s /bin/tar /bin/gtar

  touch /opt/done_with_ubuntu

  ###########
  # MINICONDA  # instead of having: From: continuumio/miniconda:4.3.11
  # from https://hub.docker.com/r/continuumio/miniconda/~/dockerfile/

  # ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

  apt-get update --fix-missing

  apt-get install -y wget bzip2 ca-certificates \
    libglib2.0-0 libxext6 libsm6 libxrender1 \
    git mercurial subversion

  #echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh
  wget --quiet https://repo.continuum.io/miniconda/Miniconda2-4.3.14-Linux-x86_64.sh -O ~/miniconda.sh
  /bin/bash ~/miniconda.sh -b -p /opt/conda
  rm ~/miniconda.sh

  #apt-get install -y curl grep sed dpkg
  #TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'`
  #curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb
  #dpkg -i tini.deb
  #rm tini.deb
  #apt-get clean

  #ENV PATH /opt/conda/bin:$PATH
  #ENTRYPOINT [ "/usr/bin/tini", "--" ]
  #CMD [ "/bin/bash" ]

  ############
  # CONDA PATH
  # this is required here as the environment section is not processed yet
  PATH=/opt/conda/bin:$PATH
  export PATH

  touch /opt/done_with_miniconda


  ##########
  # NOTEBOOK
  conda create --yes -n notebook-5.0.0 python=3.6 notebook=5.0.0
  conda env export -n notebook-5.0.0 > /opt/condaenv_notebook-5.0.0.yaml
  ############
  # CONDA PATH
  PATH="/opt/conda/envs/notebook-5.0.0/bin:$PATH"
  export PATH

  touch /opt/done_with_notebook


  ##########
  # IRKERNEL
  #wget --quiet  https://github.com/IRkernel/IRkernel/archive/0.8.8.tar.gz  -O /opt/irkermel-0.8.8.tar.gz
  #tar -xzf /opt/irkermel-0.8.8.tar.gz

  # TODO replace devtools::install_github('IRkernel/IRkernel') with
  # devtools::install_local('/opt/irkermel-0.8.8');

  # TODO MORE ALGOS ?
  # sclvm
  # ccremover

  # TODO other way to install in R:
  # /opt/conda/envs/limma/bin/R --no-restore --no-save -e \
  #   "library(devtools); install_github('IRkernel/IRkernel'); library(IRkernel); installspec(name = 'limma', displayname = 'limma-3-30.13_R 3.3');"

  # TODO how to check things out in R
  # /opt/conda/envs/basics/bin/R --no-restore --no-save -e "library(); loadedNamespaces();"


#  # BASICS 0.7.27
#  ###############
#  conda create --yes -n basics -c r \
#    r-base=3.3.2
#  #conda create -n basics-0.7.27 -c r r-base=3.3.2 r-recommended=3.3.2 r-devtools=1.12.0
#      # conda install -c bioconda bioconductor-biobase=2.34.0
#      ### downgrading     r-base r-devtools  r-recommended:     3.3.2-r3.3.2_0   to   3.3.1-r3.3.1_0
#      # conda install -c bioconda bioconductor-biocgenerics=0.20.0
#      # conda install -c r r-rcpp=0.12.8
#      ### then in R:
#      ##############
#      #library(devtools)
#      #source("http://bioconductor.org/biocLite.R")
#      #biocLite("BiocGenerics")
#      #biocLite("scran")
#      ##### install.packages("Rcpp")  ### ALREADY DONE WIH CONDA
#  conda env export -n basics > /opt/condaenv_basics.yaml
#  touch /opt/done_with_basics

#  MODULE_NAME="basics"
#  MODULE_VERSION="0.7.27"
#  MODULE_DESCRIPTION=""
#  MODULE_CHANNELS="-c r"
#  MODULE_PACKAGES="r-devtools=1.12.0 r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0"
#  MODULE_RBASEVERSION="rbase-3.3.2"
#  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
#  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
#  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
#    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
#  touch /opt/done_with_${MODULE_NAME}

#     COMBATPY 0.0.1_20170804
#    ########################
##    conda create --yes -n combatpy-0.0.1_20170804  -c bioconda -c r \
##      bioconductor-sva=3.20.0 \
##      bioconductor-biocinstaller=1.24.0 \
##      python=2.7 \
##      pandas=0.20.3 \
##      patsy=0.4.1
#  conda create --yes -n combatpy  -c bioconda -c r \
#    bioconductor-sva=3.20.0 \
#    bioconductor-biocinstaller=1.24.0 \
#    r-devtools \
#    r-irkernel \
#    pandas \
#    patsy \
#    python=3 \
#    jupyter
#  conda env export -n combatpy > /opt/condaenv_combatpy-0.0.20170804_0.yaml
#  /opt/conda/envs/combatpy/bin/R --no-restore --no-save -e \
#    "library(BiocInstaller); biocLite('bladderbatch')"
#  /opt/conda/envs/combatpy/bin/R --no-restore --no-save -e \
#    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'combatpy', displayname = 'combatpy_R 3.3');"
#  # #git clone https://github.com/brentp/combat.py.git /opt/members/combatpy/repo/
#  touch /opt/done_with_combatpy
  MODULE_NAME="combatpy"
  MODULE_VERSION="0.0.1_20170804"
  MODULE_DESCRIPTION=""
  MODULE_CHANNELS="-c bioconda -c r"
  MODULE_PACKAGES="bioconductor-sva=3.20.0 bioconductor-biocinstaller=1.24.0 r-devtools=1.12.0 r-irkernel=0.7.1 pandas patsy python=3.6.2 jupyter=1.0.0"
  MODULE_RBASEVERSION="rbase-3.3"
  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
  ###
  /opt/conda/envs/combatpy/bin/R --no-restore --no-save -e "library(BiocInstaller); biocLite('bladderbatch')"
  ###
  touch /opt/done_with_${MODULE_NAME}

  MODULE_NAME="limma"
  MODULE_VERSION="3.30.13"
  MODULE_DESCRIPTION="Linear Models for Microarray and RNA-Seq Data"
  MODULE_CHANNELS="-c bioconda -c r"
  MODULE_PACKAGES="r-devtools=1.12.0 r-irkernel=0.7.1 bioconductor-limma=3.30.13 python=3.6.2 jupyter=1.0.0"
  MODULE_RBASEVERSION="rbase-3.3.2"
  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
  touch /opt/done_with_${MODULE_NAME}

  MODULE_NAME="ruvseq"
  MODULE_VERSION="1.8.0"
  MODULE_DESCRIPTION="Remove Unwanted Variation from RNA-Seq Data"
  MODULE_CHANNELS="-c bioconda -c pjones -c r"
  MODULE_PACKAGES="bioconductor-edger=3.16.5 bioconductor-edaseq=2.8.0 bioconductor-ruvseq=1.8.0 r-devtools=1.11.1 r-irkernel==0.7 python=3.6.2 jupyter=1.0.0"
  MODULE_RBASEVERSION="rbase-3.3.1"
  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
  touch /opt/done_with_${MODULE_NAME}

  MODULE_NAME="scnorm"
  MODULE_VERSION="0.99.7"
  MODULE_DESCRIPTION="robust normalization of single-cell RNA-seq data"
  MODULE_CHANNELS="-c bioconda -c r -c kurtwheeler"
  MODULE_PACKAGES="r-argparse=1.0.4 r-devtools=1.13.2 r-irkernel=0.7.1 bioconductor-biocinstaller=1.26.0 python=3.6.2 jupyter=1.0.0"
  MODULE_RBASEVERSION="rbase-3.4.1"
  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
  touch /opt/done_with_${MODULE_NAME}

  MODULE_NAME="scran"
  MODULE_VERSION="1.4.5"
  MODULE_DESCRIPTION="Implements a variety of low-level analyses of single-cell RNA-seq data."
  MODULE_CHANNELS="-c r -c bioconda "
  MODULE_PACKAGES="r-xml=3.98_1.5 r-httpuv=1.3.3 r-shiny=0.14.2 r-shinydashboard=0.5.3 r-devtools=1.12.0 r-irkernel=0.7.1 bioconductor-biocinstaller=1.24.0 bioconductor-biomart=2.28.0 python=3.6.2 jupyter=1.0.0"
  MODULE_RBASEVERSION="rbase-3.3.2"
  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
  touch /opt/done_with_${MODULE_NAME}

  MODULE_NAME="seurat"
  MODULE_VERSION="2.0.0"
  MODULE_DESCRIPTION=""
  MODULE_CHANNELS="-c r"
  MODULE_PACKAGES="r-devtools=1.13.2 r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0"
  MODULE_RBASEVERSION="rbase-3.4.1"
  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
  ###
  /opt/conda/envs/seurat/bin/R --no-restore --no-save -e "devtools::install_github('satijalab/seurat');"
  ###
  touch /opt/done_with_${MODULE_NAME}

  MODULE_NAME="svaseq"
  MODULE_VERSION="1.8.0"
  MODULE_DESCRIPTION=""
  MODULE_CHANNELS="-c bioconda -c r"
  MODULE_PACKAGES="bioconductor-sva=3.20.0 r-devtools=1.12.0 r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0"
  MODULE_RBASEVERSION="rbase-3.3"
  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
  touch /opt/done_with_${MODULE_NAME}

  MODULE_NAME="vamf"
  MODULE_VERSION="0.0.20170804"
  MODULE_DESCRIPTION=""
  MODULE_CHANNELS="-c bioconda -c r"
  MODULE_PACKAGES="bioconductor-biocinstaller=1.24.0 r-devtools=1.12.0 r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0"
  MODULE_RBASEVERSION="rbase-3.3"
  conda create --yes -n ${MODULE_NAME} ${MODULE_CHANNELS} ${MODULE_RBASEVERSION} ${MODULE_PACKAGES}
  conda env export -n ${MODULE_NAME} > /opt/condaenv_${MODULE_NAME}_0.yaml
  /opt/conda/envs/${MODULE_NAME}/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = '${MODULE_NAME}', displayname = '${MODULE_NAME}-${MODULE_VERSION}_${MODULE_RBASEVERSION}');"
  touch /opt/done_with_${MODULE_NAME}

  touch /opt/done_with_members_envs

  ########################
  # CONDA EXPORT AND CLEAN

  # irkernel cleanup
  # rm /opt/irkermel-0.8.8.tar.gz

  /opt/conda/bin/conda env export -n root > /opt/condaenv_root_`date +%Y-%m-%d-%H-%M`.yaml

  # cleanup ???M
  /opt/conda/bin/conda clean --index-cache --tarballs --packages --yes

  touch /opt/done_with_conda_export_clean

  chmod +755 -R /opt/*

  set +x

  #############################################################################

%environment

  PATH="/opt/conda/bin:$PATH"
  PATH="/opt/conda/envs/notebook-5.0.0/bin:$PATH"
  PATH="/opt/members/notebook:$PATH"

  PATH="/opt/members/basics:$PATH"
  PATH="/opt/members/combatpy:$PATH"
  PATH="/opt/members/limma:$PATH"
  PATH="/opt/members/ruvseq:$PATH"
  PATH="/opt/members/scran:$PATH"
  PATH="/opt/members/seurat:$PATH"
  PATH="/opt/members/scnorm:$PATH"
  PATH="/opt/members/svaseq:$PATH"
  PATH="/opt/members/vamf:$PATH"

  export PATH

  HOSTIP=$(hostname -i)
  export HOSTIP

  alias echopathtr='echo $PATH | tr ":" "\n"'
  alias ll='ls -lhF'

 ##############################################################################

%labels

  MAINTAINER alaindomissy@gmail.com
  VERSION 0.0.0-20170824
  BUILD_DATE "${date -Iminutes}"

  #############################################################################

%files

  members/*          /opt/
  documentation      /opt/
  tests              /opt/

  #############################################################################

%runscript

  # this will get copied to /.singularity.d/runscript indide the container
  # which will run whenever the container is called as an executable

  set -o xtrace
  set -o nounset
  set -o errexit
  #set -o pipefail

  if [ $# -eq 0 ]
  then

      # /opt/wf/eclip

      IDATE=$(date -Iseconds)
      # IDATE=$(date -Iseconds | tr "\:" "-" | tr "T" "+")

      IMAGENAME=scbatch_${IDATE}_${SINGULARITY_NAME}
      mv ${SINGULARITY_NAME} ${IMAGENAME}
      ln -sf ${IMAGENAME} scbatch.img

      cp /opt/patches/scripts/*   ./
      mv ./scbatchrc              ./.scbatchrc

      mkdir -p commands
      mv scbatch ./commands/
      cd commands
      ln -sf scbatch scbatch_notebook
      ln -sf scbatch scbatch_password

      ln -sf scbatch scbatch_basics
      ln -sf scbatch scbatch_combatpy
      ln -sf scbatch scbatch_limma
      ln -sf scbatch scbatch_ruvseq
      ln -sf scbatch scbatch_scnorm
      ln -sf scbatch scbatch_scran
      ln -sf scbatch scbatch_seurat
      ln -sf scbatch scbatch_svaseq
      ln -sf scbatch scbatch_vamf

      ln -sf scbatch scbatch_basics_notebook
      ln -sf scbatch scbatch_combatpy_notebook
      ln -sf scbatch scbatch_limma_notebook
      ln -sf scbatch scbatch_ruvseq_notebook
      ln -sf scbatch scbatch_scnorm_notebook
      ln -sf scbatch scbatch_scran_notebook
      ln -sf scbatch scbatch_seurat_notebook
      ln -sf scbatch scbatch_svaseq_notebook
      ln -sf scbatch scbatch_vamf_notebook

      cd -

      echo
      echo ======================
      echo please type:
      echo
      echo        source .scbatchrc
      echo
      echo and enjoy scbatch!
      echo ======================
      echo

  else
      echo "scbatch image called with run and some arguments - did you mean to exec instead ?"
  fi
###############################################################################
%test

  # /opt/tests/test
