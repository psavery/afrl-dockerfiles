FROM library/ubuntu:18.04 AS build
MAINTAINER Kitware <kitware@kitware.com>

SHELL ["/bin/bash", "-c"]

RUN apt-get update && apt-get -y upgrade && \
  apt-get -y install wget

RUN CMAKE_NAME=cmake-3.11.3-Linux-x86_64 && \
  wget https://cmake.org/files/v3.11/$CMAKE_NAME.tar.gz && \
  tar -xzvf $CMAKE_NAME.tar.gz && \
  pushd . && \
  cd $CMAKE_NAME && \
  cp -r bin /usr/ && \
  cp -r share /usr/ && \
  cp -r doc /usr/share/ && \
  cp -r man /usr/share/ && \
  popd && \
  rm -rf $CMAKE_NAME $CMAKE_NAME.tar.tgz

RUN apt-get -y install \
  git \
  build-essential \
  libtbb-dev \
  python3-dev \
  qt5-default \
  qtwebengine5-dev \
  libqt5svg5-dev \
  qttools5-dev

# Build the Dream3D SDK
RUN mkdir -p src && \
  pushd . && \
  cd src && \
  git clone https://github.com/bluequartzsoftware/dream3dsuperbuild && \
  cd dream3dsuperbuild && \
  git checkout 8773ca90 && \
  popd && \
  mkdir -p build && \
  cd build && \
  export WORKINGDIR=$PWD && \
  mkdir -p dream3d-super && \
  cd dream3d-super && \
  cmake ../../src/dream3dsuperbuild \
    -DDREAM3D_SDK=$WORKINGDIR/dream3d-sdk \
    -DINSTALL_QT5=OFF \
    -DQMAKE_EXECUTABLE=/usr/bin/qmake \
    -DBUILD_ITK=OFF \
    -DCMAKE_BUILD_TYPE=Release && \
  make -j "$(nproc)"

# Clone dream3d, select plugins, and all required repositories
RUN cd src && \
# Get dream3d
  git clone https://github.com/BlueQuartzSoftware/DREAM3D && \
  cd DREAM3D && git checkout d0d97069 && cd .. && \
# Get required repositories
  git clone https://github.com/BlueQuartzSoftware/CMP && \
  cd CMP && git checkout 258ab4b8 && cd .. && \
  git clone https://github.com/BlueQuartzSoftware/SIMPL && \
  cd SIMPL && git checkout c802b1bd && cd .. && \
  git clone https://github.com/BlueQuartzSoftware/SIMPLView && \
  cd SIMPLView && git checkout dd86fcb8 && cd .. && \
# Get desired plugins
  mkdir -p DREAM3D_Plugins && \
  cd DREAM3D_Plugins && \
# Anisotropy
#  git clone https://github.com/dream3d/Anisotropy && \
#  cd Anisotropy && git checkout bda11716 && cd .. && \
# FiberToolbox
  git clone https://github.com/dream3d/FiberToolbox && \
  cd FiberToolbox && git checkout 01dbbc92 && cd .. && \
# MASSIFUtilities
  git clone https://github.com/dream3d/MASSIFUtilities && \
  cd MASSIFUtilities && git checkout c01e47ef && cd .. && \
# DDDAnalysisToolbox
  git clone https://github.com/dream3d/DDDAnalysisToolbox && \
  cd DDDAnalysisToolbox && git checkout 76d737cd && cd .. && \
# HEDMAnalysis
  git clone https://github.com/dream3d/HEDMAnalysis && \
  cd HEDMAnalysis && git checkout 99abd70b && cd .. && \
# TransformationPhase
  git clone https://github.com/dream3d/TransformationPhase && \
  cd TransformationPhase && git checkout a2c686cb && cd .. && \
# DREAM3DReview
  git clone https://github.com/dream3d/DREAM3DReview && \
  cd DREAM3DReview && git checkout 2f86e8f8 && cd .. && \
# ImageProcessing
#  git clone https://github.com/dream3d/ImageProcessing && \
#  cd ImageProcessing && git checkout 5c1cff27 && cd .. && \
# UCSBUtilities
  git clone https://github.com/dream3d/UCSBUtilities && \
  cd UCSBUtilities && git checkout acb1d502 && cd ..

RUN export SRCDIR=$PWD/src && \
    export DREAM3D_EXTRA_PLUGINS="FiberToolbox;MASSIFUtilities;DDDAnalysisToolbox;HEDMAnalysis;TransformationPhase;DREAM3DReview;UCSBUtilities" && \
  cd build && \
  mkdir -p dream3d-build && \
  cd dream3d-build && \
# We use an || here because the first cmake command will fail for sure,
# but we must run cmake twice. The second time should pass.
  (cmake $SRCDIR/DREAM3D \
    -DDREAM3D_SDK=/build/dream3d-sdk \
    -DCMP_SOURCE_DIR=$SRCDIR/CMP \
    -DSIMPLProj_SOURCE_DIR=$SRCDIR/SIMPL \
    -DSIMPLViewProj_SOURCE_DIR=$SRCDIR/SIMPLView \
    -DSIMPLView_BUILD_DOCUMENTATION=OFF \
    -DBUILD_TESTING=OFF \
    -DSIMPL_BUILD_TESTING=OFF \
    -DSIMPL_USE_ITK=OFF \
    -DCMAKE_BUILD_TYPE=Release || \
   cmake $SRCDIR/DREAM3D -DDREAM3D_EXTRA_PLUGINS=$DREAM3D_EXTRA_PLUGINS)

RUN cd build/dream3d-build && \
  make -j "$(nproc)"

## Create a new image
FROM library/ubuntu:18.04

# Install runtime deps
RUN apt-get update && apt-get -y install \
  libtbb2 \
  libqt5core5a \
  libqt5network5 \
# Unfortunately, libqt5gui5 is huge (~300MB), and we only need it because we
# use "Generate IPF Colors" from the Orientation Analysis plugin in Dream3D,
# and the Orientation Analysis plugin depends on libqt5gui5.
# It would be very nice if we removed this dependency in the future.
  libqt5gui5

COPY --from=build /build/dream3d-build/Bin /usr/local/dream3d/Bin

RUN cd /usr/local/dream3d/Bin && \
  rm -r Data && \
# The environment setup
  echo "export PATH=/usr/local/dream3d/Bin:\$PATH" >> ~/setupEnvironment && \
  echo "export LD_LIBRARY_PATH=/usr/local/dream3d/Bin" >> ~/setupEnvironment && \
# The command we will run
  echo "source ~/setupEnvironment" >> ~/runPipelineRunner && \
  echo "PipelineRunner -p \$1" >> ~/runPipelineRunner

ENTRYPOINT ["bash", "/root/runPipelineRunner"]
