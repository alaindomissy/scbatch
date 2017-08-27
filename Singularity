BootStrap: docker
From:  ubuntu:16.04

  ####
%setup
  ####
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

  mkdir -p $SINGULARITY_ROOTFS/opt/condaenv
  mkdir -p $SINGULARITY_ROOTFS/opt/donewith
  mkdir -p $SINGULARITY_ROOTFS/opt/members
  mkdir -p $SINGULARITY_ROOTFS/opt/patches

  # these are needed early for post section
  cp -r ./members/* $SINGULARITY_ROOTFS/opt/members/
  cp -r ./patches/* $SINGULARITY_ROOTFS/opt/patches/

  ###
%post
  ###
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

  touch /opt/donewith/ubuntu

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

  # this is required here as the environment section is not processed yet
  PATH=/opt/conda/bin:$PATH
  export PATH
  echo ************************************************************************
  touch /opt/donewith/conda
  echo ************************************************************************


  echo ************************************************************************
  NAME=notebook
  VERSION=5.0.0
  echo ************************************************************************
  conda create --yes -n notebook python=3.6 notebook=5.0.0
  conda env export -n notebook > /opt/condaenv/notebook-5.0.0.yaml
  PATH="/opt/conda/envs/notebook/bin:$PATH"
  export PATH
  NOTEBOOKKERNELS=/opt/conda/envs/notebook/share/jupyter/kernels/
  echo ************************************************************************
  touch /opt/donewith/notebook
  echo ************************************************************************



  # Todo IRKERNEL
  #wget --quiet  https://github.com/IRkernel/IRkernel/archive/0.8.8.tar.gz  -O /opt/irkermel-0.8.8.tar.gz
  #tar -xzf /opt/irkermel-0.8.8.tar.gz

  # TODO replace devtools::install_github('IRkernel/IRkernel') with
  # devtools::install_local('/opt/irkermel-0.8.8');

  # TODO MORE ALGOS ?
  # ccremover

  # TODO other way to install in R:
  # /opt/conda/envs/limma/bin/R --no-restore --no-save -e \
  #   "library(devtools); install_github('IRkernel/IRkernel'); library(IRkernel); installspec(name = 'limma', displayname = 'limma-3-30.13_R 3.3');"

  # TODO how to check things out in R
  # /opt/conda/envs/basics/bin/R --no-restore --no-save -e "library(); loadedNamespaces();"


  add_algorithm()
  {
    echo **********************************************************************
    NAME=$1
    VERSION=$2
    echo **********************************************************************
    RVERSION=$3
    DESCRIPTION=$4
    CHANNELS=$5
    PACKAGES=$6
    PIPS=$7
    BIOCLITE=$8
    GITHUB=$9
    URL=${10}
    ENVBIN="/opt/conda/envs/${NAME}/bin"
    REXEC="${ENVBIN}/R --no-restore --no-save -e"
    RBASEPACKAGE=""
    if [ ${RVERSION} != "none" ] ; then
      RBASEPACKAGE="r-base=${RVERSION}"
    fi
    conda create --yes --name ${NAME} ${CHANNELS} ${RBASEPACKAGE} ${PACKAGES}
    conda env export --name ${NAME} > /opt/condaenv/${NAME}.yaml
    if [ ${RVERSION} != "none" ] ; then
      ${REXEC} "devtools::install_github('IRkernel/IRkernel'); IRkernel::installspec(name='${NAME}',displayname='${NAME}-${VERSION}_rbase-${RVERSION}');"
      cp -r ${ENVBIN}/share/jupyter/kernels/${NAME} ${NOTEBOOKKERNELS}/
    fi
    if [ ${PIPS} != "none" ]    ; then ${ENVBIN}/pip install "${PIPS}"                  ; fi
    if [ ${BIOCLITE} != "none" ]; then ${REXEC} "BiocInstaller::biocLite('${BIOCLITE}')"; fi
    if [ ${GITHUB} != "none" ]  ; then ${REXEC} "devtools::install_github('${GITHUB}')" ; fi
    if [ ${URL} != "none" ]     ; then ${REXEC} "devtools::install_url('${URL}')"       ; fi
    echo **********************************************************************
    touch /opt/donewith/${NAME}
    echo **********************************************************************
  }


  # r-irkernel=0.7.1 jupyter=1.0.0
  add_algorithm \
    combatpy \
    0.0.1_20170804 \
    3.3 \
    "Combatpy." \
    "-c bioconda -c r" \
    "r-argparse=1.0.4 r-devtools=1.12.0 bioconductor-biocinstaller=1.24.0 bioconductor-sva=3.20.0 pandas=0.20.3 patsy=0.4.1 jupyter_client=5.1.0 python=3.6.2" \
    "none" \
    "bladderbatch" \
    "none" \
    "none"

  # conda install -c chasehere r-rpython
  # conda install -c bioconda limix
  # conda install -c conda-forge gpy
  # "scipy h5py numpy matplotlib scikit-learn re python=2.7 jupyter"
  #add_algorithm fsclvm 1.0.0.dev10 3.2.2
  # r-irkernel jupyter=1.0.0"
  add_algorithm \
    fsclvm \
    1.0.0.dev10 \
    3.3.2 \
    "Scalable modelling framework for single-cell RNA-seq data that uses gene set annotations to dissect single-cell transcriptome heterogeneity, thereby allowing to identify biological drivers of cell-to-cell variability and model confounding factors." \
    "-c defaults" \
    "r-argparse=1.0.4 scipy=0.19.1 h5py=2.7.0 numpy=1.13.1 matplotlib=2.0.2 scikit-learn=0.19.0 jupyter_client=5.1.0 python=2.7.13" \
    "fscLVM==1.0.0.dev10" \
    none \
    none \
    none
    # devtools::install_github('PMBio/scLVM');
    # https://github.com/PMBio/scLVM/archive/V0.1.tar.gz

  # r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0

  add_algorithm \
    limma \
    3.30.13 \
    3.3.2 \
    "Linear Models for Microarray and RNA-Seq Data" \
    "-c bioconda -c r" \
    "r-argparse=1.0.4 r-devtools=1.12.0 bioconductor-limma=3.30.13 jupyter_client=5.1.0" \
    none \
    none \
    none \
    none

  # r-irkernel==0.7 python=3.6.2 jupyter=1.0.0
  add_algorithm \
    ruvseq \
    1.8.0 \
    3.3.1 \
    "Remove Unwanted Variation from RNA-Seq Data" \
    "-c bioconda -c pjones -c r" \
    "r-argparse=1.0.1 r-devtools=1.11.1 bioconductor-edger=3.16.5 bioconductor-edaseq=2.8.0 bioconductor-ruvseq=1.8.0 jupyter_client=5.1.0" \
    none \
    none \
    none \
    none

#  # conda install -c chasehere r-rpython
#  # conda install -c bioconda limix
#  # conda install -c conda-forge gpy
#  #r-argparse
#  # r-irkernel=0.7
#  add_algorithm sclvm 0.1.8 3.2.2 \
#    "Modelling framework for single-cell RNA-seq data that can be used to dissect the observed heterogeneity into different sources, thereby allowing for the correction of confounding sources of variation." \
#    "-c r -c bioconda -c chasehere -c conda-forge" \
#    "r-irkernel hdf5 h5py=2.7.0 matplotlib=2.0.2 gpy=1.7.7 limix=0.7.12 r-rpython=0.0_6 python=2.7.13 jupyter=1.0.0" \
#    "scLVM==0.1.8" \
#    "none" \
#    "none" \
#    "none"
#    #'/opt/conda/envs/seurat/bin/R --no-restore --no-save -e "devtools::install_github('PMBio/scLVM')";'
#    # https://github.com/PMBio/scLVM/archive/V0.1.tar.gz

  # r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0
  add_algorithm \
    scnorm \
    0.99.7 \
    rbase-3.4.1 \
    "Robust normalization of single-cell RNA-seq data." \
    "-c bioconda -c r -c kurtwheeler" \
    "r-argparse=1.0.4 r-devtools=1.13.2 bioconductor-biocinstaller=1.26.0 jupyter_client=5.1.0" \
    none \
    none \
    none \
    "https://bioconductor.org/packages/devel/bioc/src/contrib/SCnorm_0.99.7.tar.gz"

  #  r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0
  add_algorithm \
    scran \
    1.4.5 \
    3.3.2 \
    "Implements a variety of low-level analyses of single-cell RNA-seq data." \
    "-c r -c bioconda " \
    "r-argparse=1.0.4 r-devtools=1.12.0 bioconductor-biocinstaller=1.24.0 r-xml=3.98_1.5 r-httpuv=1.3.3 r-shiny=0.14.2 r-shinydashboard=0.5.3 bioconductor-biomart=2.28.0 jupyter_client=5.1.0" \
    none \
    "scran" \
    none \
    none

#  # r-irkernel=0.7.1
#  # r-rcpp=0.12.11
#  # "r-argparse=1.0.4 r-devtools=1.12.0 bioconductor-biocinstaller=1.24.0 r-rcpp=0.12.8 bioconductor-biocgenerics=0.20.0 python=3.6.2 jupyter=1.0.0"
#  add_algorithm basics 0.7.27 3.3.2 \
#    "Bayesian Analysis of Single-Cell Sequencing Data." \
#    "-c r -c bioconda" \
#    "r-argparse=1.0.4 r-devtools=1.12.0 bioconductor-biocinstaller=1.24.0  r-xml=3.98_1.5 r-httpuv=1.3.3 r-shiny=0.14.2 r-shinydashboard=0.5.3 bioconductor-biomart=2.28.0 r-rcpp=0.12.8 bioconductor-biocgenerics=0.20.0 python=3.6.2 jupyter_client=5.1.0" \
#    "none" \
#    "scran" \
#    "catavallejos/BASiCS" \
#    "none"

  # r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0
  add_algorithm \
    seurat \
    2.0.0 \
    3.4.1 \
    "Seurat." \
    "-c r" \
    "r-argparse=1.0.4 r-devtools=1.13.2 jupyter_client=5.1.0" \
    none \
    none \
    "satijalab/seurat" \
    none

  # r-irkernel=0.7.1 python=3.6.2 jupyter=1.0.0
  add_algorithm \
    svaseq \
    1.8.0 \
    3.3 \
    "Svaseq." \
    "-c bioconda -c r" \
    "r-argparse=1.0.4 r-devtools=1.12.0 bioconductor-sva=3.20.0 jupyter_client=5.1.0" \
    none \
    none \
    none \
    none

  # r-irkernel=0.7.1  python=3.6.2 jupyter=1.0.0
  add_algorithm \
    vamf \
    0.0.20170804 \
    3.3 \
    "Vamf." \
    "-c bioconda -c r" \
    "r-argparse=1.0.4 r-devtools=1.12.0 bioconductor-biocinstaller=1.24.0 jupyter_client=5.1.0" \
    none \
    none \
    none \
    none


  touch /opt/donewith/members_envs

  # irkernel cleanup
  # rm /opt/irkermel-0.8.8.tar.gz
  /opt/conda/bin/conda env export -n root > /opt/condaenv/root_$(date +%Y-%m-%d-%H-%M).yaml
  # cleanup ???M
  /opt/conda/bin/conda clean --index-cache --tarballs --packages --yes
  chmod --recursive --changes +755 /opt/*

  touch /opt/donewith/conda_export_clean_chmod

  set +x


%environment

  PATH="/opt/conda/bin:$PATH"
  PATH="/opt/conda/envs/notebook/bin:$PATH"

  PATH="/opt/members/notebook:$PATH"
  PATH="/opt/members/rargparse:$PATH"
  PATH="/opt/members/scbatchget:$PATH"
  #PATH="/opt/members/scbatchdataset:$PATH"

  PATH="/opt/members/all:$PATH"
  PATH="/opt/members/basics:$PATH"
  PATH="/opt/members/combatpy:$PATH"
  PATH="/opt/members/fsclvm:$PATH"
  PATH="/opt/members/limma:$PATH"
  PATH="/opt/members/ruvseq:$PATH"
  PATH="/opt/members/sclvm:$PATH"
  PATH="/opt/members/scnorm:$PATH"
  PATH="/opt/members/scran:$PATH"
  PATH="/opt/members/seurat:$PATH"
  PATH="/opt/members/svaseq:$PATH"
  PATH="/opt/members/vamf:$PATH"

  export PATH

  HOSTIP=$(hostname -i)
  export HOSTIP

  alias echopathtr='echo $PATH | tr ":" "\n"'
  alias ll='ls -lhF'

 ######
%labels
 ######

  MAINTAINER alaindomissy@gmail.com
  VERSION 0.0.1-20170826
  BUILD_DATE "${date -Iminutes}"

  ####
%files
  ####
  members/*          /opt/
  documentation      /opt/
  tests              /opt/

  ########
%runscript
  ########
  # this will get copied to /.singularity.d/runscript indide the container
  # which will run whenever the container is called as an executable

  set -o xtrace
  set -o nounset
  set -o errexit
  #set -o pipefail

  if [ $# -eq 0 ]
  then

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
      ln -sf scbatch scbatch_reset_password
      ln -sf scbatch scbatch_reset_config
      ln -sf scbatch scbatch_getdataset
      #ln -sf scbatch scbatch_getreference

      ln -sf scbatch scbatch_basics
      ln -sf scbatch scbatch_combatpy
      ln -sf scbatch scbatch_fsclvm
      ln -sf scbatch scbatch_limma
      ln -sf scbatch scbatch_ruvseq
      ln -sf scbatch scbatch_sclvm
      ln -sf scbatch scbatch_scnorm
      ln -sf scbatch scbatch_scran
      ln -sf scbatch scbatch_seurat
      ln -sf scbatch scbatch_svaseq
      ln -sf scbatch scbatch_vamf

      ln -sf scbatch scbatch_all_notebook

      ln -sf scbatch scbatch_basics_notebook
      ln -sf scbatch scbatch_combatpy_notebook
      ln -sf scbatch scbatch_fsclvm_notebook
      ln -sf scbatch scbatch_limma_notebook
      ln -sf scbatch scbatch_ruvseq_notebook
      ln -sf scbatch scbatch_sclvm_notebook
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
