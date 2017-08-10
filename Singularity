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


  ########
  # BASICS 0.7.27
  conda create --yes -n basics -c r \
    r-base=3.3.2
  #conda create -n basics-0.7.27 -c r r-base=3.3.2 r-recommended=3.3.2 r-devtools=1.12.0
      # conda install -c bioconda bioconductor-biobase=2.34.0
      ### downgrading     r-base r-devtools  r-recommended:     3.3.2-r3.3.2_0   to   3.3.1-r3.3.1_0
      # conda install -c bioconda bioconductor-biocgenerics=0.20.0
      # conda install -c r r-rcpp=0.12.8
      ### then in R:
      ##############
      #library(devtools)
      #source("http://bioconductor.org/biocLite.R")
      #biocLite("BiocGenerics")
      #biocLite("scran")
      ##### install.packages("Rcpp")  ### ALREADY DONE WIH CONDA

  conda env export -n basics > /opt/condaenv_basics.yaml

  #/opt/conda/envs/basics/bin/R --no-restore --no-save -e "library(); loadedNamespaces();"

  touch /opt/done_with_basics

  ##########
  # COMBATPY 0.0.1_20170804
  #conda create --yes -n combatpy-0.0.1_20170804  -c bioconda -c r \
  #  bioconductor-sva=3.20.0 \
  #  bioconductor-biocinstaller=1.24.0 \
  #  python=2.7 \
  #  pandas=0.20.3 \
  #  patsy=0.4.1
  conda create --yes -n combatpy  -c bioconda -c r \
    bioconductor-sva=3.20.0 \
    bioconductor-biocinstaller=1.24.0 \
    r-devtools \
    r-irkernel \
    pandas \
    patsy \
    python=3 \
    jupyter
  conda env export -n combatpy > /opt/condaenv_combatpy-0.0.20170804_0.yaml

  /opt/conda/envs/combatpy/bin/R --no-restore --no-save -e \
    "library(BiocInstaller); biocLite('bladderbatch')"
  /opt/conda/envs/combatpy/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'combatpy', displayname = 'combatpy_R 3.3');"

  #git clone https://github.com/brentp/combat.py.git /opt/members/combatpy/repo/

  touch /opt/done_with_combatpy

  #######
  # LIMMA 3.30.13
  conda create --yes -n limma -c bioconda -c r \
    r-devtools=1.12.0 \
    r-irkernel=0.7.1 \
    bioconductor-limma=3.30.13 \
    python=3 \
    jupyter
  conda env export -n limma > /opt/condaenv_limma_0.yaml

  #/opt/conda/envs/limma/bin/R --no-restore --no-save -e "library(limma);"
  # "library(); library(limma); loadedNamespaces();"

  /opt/conda/envs/limma/bin/R --no-restore --no-save -e \
    "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'limma_ir33', displayname = 'limma-3-30.13_R 3.3');"
  /opt/conda/envs/limma/bin/R --no-restore --no-save -e \
     "library(devtools); install_github('IRkernel/IRkernel'); library(IRkernel); installspec(name = 'limma', displayname = 'limma-3-30.13_R 3.3');"

  touch /opt/done_with_limma

  ########
  # RUVSEQ 1.8.0
  conda create --yes -n ruvseq -c bioconda -c pjones -c r \
    bioconductor-edger=3.16.5 \
    bioconductor-edaseq=2.8.0 \
    bioconductor-ruvseq=1.8.0 \
    r-devtools=1.11.1 \
    r-irkernel==0.7
  conda env export -n ruvseq > /opt/condaenv_ruvseq_0.yaml

  #/opt/conda/envs/ruvseq/bin/R --no-restore --no-save -e "library(); library(RUVSeq); loadedNamespaces()"
  /opt/conda/envs/ruvseq/bin/R --no-restore --no-save -e \
     "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'ruvseq', displayname = 'ruvseq_R 3.3');"

  touch /opt/done_with_ruvseq

  ########
  # SCNORM
  conda create --yes -n scnorm-0.7.27 -c bioconda -c r \
    bioconductor-biocinstaller=1.24.0 \
    r-devtools=1.12.0 \
    r-irkernel=0.7.1
  conda env export -n scnorm-0.7.27 > /opt/condaenv_scnorm-0.0.20170804_0.yaml

  /opt/conda/envs/scnorm-0.7.27/bin/R --no-restore --no-save -e \
     "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'scnorm', displayname = 'scnorm-0.7.27_R 3.3');"

  touch /opt/done_with_scnorm

  ########
  # SCRAN
  #conda create --yes -n scran-1.4.5 -c r -c bioconda \
  #   r-base=3.4.1 \
  #   r-xml=3.98_1.7 \
  #   r-httpuv=1.3.3 \
  #   r-shiny=1.0.3 \
  #   r-shinydashboard=0.6.0 \
  #   bioconductor-biocinstaller=1.24.0  \
  #   bioconductor-biomart=2.28.0 \
  #   r-irkernel=0.7.1
  conda create --yes -n scran-1.4.5 -c r -c bioconda \
     r-base=3.3.2 \
     r-xml=3.98_1.5 \
     r-httpuv=1.3.3 \
     r-shiny \
     r-shinydashboard \
     r-devtools \
     r-irkernel=0.7.1 \
     bioconductor-biocinstaller  \
     bioconductor-biomart
  conda env export -n scran-1.4.5 > /opt/condaenv_scran-1.4.5_0.yaml

  /opt/conda/envs/scran-1.4.5/bin/R --no-restore --no-save -e \
     "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'scran', displayname = 'scran-1.4.5_R 3.3');"

  touch /opt/done_with_scran

  ########
  # SEURAT
  conda create --yes -n seurat-2.0.0 -c r \
    r-devtools=1.13.2 \
    r-irkernel=0.7.1
  conda env export -n seurat-2.0.0 > /opt/condaenv_seurat-2.0.0_0.yaml

  /opt/conda/envs/seurat-2.0.0/bin/R --no-restore --no-save -e "devtools::install_github('satijalab/seurat');"
  /opt/conda/envs/seurat-2.0.0/bin/R --no-restore --no-save -e \
     "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'seurat', displayname = 'seurat-2.0.0_R 3.4')"

  touch /opt/done_with_seurat

  ########
  # SVASEQ 3.20.0
  conda create --yes -n svaseq -c bioconda -c r \
    bioconductor-sva=3.20.0 \
    r-devtools=1.12.0 \
    r-irkernel=0.7.1
  conda env export -n svaseq > /opt/condaenv_svaseq_0.yaml

  # /opt/conda/envs/svaseq-1.8.0/bin/R --no-restore --no-save -e "library(); library(RUVSeq); loadedNamespaces(); q()"
  /opt/conda/envs/svaseq-1.8.0/bin/R --no-restore --no-save -e \
     "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'svaseq', displayname = 'svaseq_R 3.3')"

  touch /opt/done_with_svaseq

  ##########
  # VAMF
  conda create --yes -n vamf -c bioconda -c r \
    bioconductor-biocinstaller=1.24.0 \
    r-devtools \
    r-irkernel=0.7.1
  conda env export -n vamf > /opt/condaenv_vamf-0.0.20170804_0.yaml

  /opt/conda/envs/vamf/bin/R --no-restore --no-save -e \
     "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name = 'vamf', displayname = 'vamf_R 3.3')"

  #git clone https://github.com/willtownes/vamf-paper.git  /opt/members/vamf/repo/

  touch /opt/done_with_vamf

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
  VERSION 0.0.0-20170809
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
      #ln -sf scbatch scbatch_password

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
      echo
  fi
###############################################################################
%test

  # /opt/tests/test
