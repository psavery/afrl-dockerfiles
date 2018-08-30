FROM library/ubuntu:18.04 AS build
LABEL Author="BlueQuartz Software <info@bluequartz.net>"

SHELL ["/bin/bash", "-c"]

RUN apt-get update && \
  apt-get -y upgrade && \
  apt-get -y install software-properties-common && \
  add-apt-repository ppa:beineri/opt-qt-5.11.1-bionic && \
  apt-get update && \
  apt-get -y install \
  qt511base \
  qt511x11extras \
  qt511svg \
  qt511tools \
  wget \
  git \
  build-essential \
  fontconfig \
  dbus \
  libx11-6 \
  libx11-xcb1 \
  libgl1-mesa-dev \
  libpython3.5 \
  python3 \
  python3-distutils

# Define the DREAM3D_SDK directory
ENV DREAM3D_SDK=/opt/DREAM3D_SDK

# Define the version of CMake that we will be using
ENV CMAKE_VERSION=3.11
ENV CMAKE_PATCH=3
ENV CMAKE_NAME=cmake-$CMAKE_VERSION.$CMAKE_PATCH-Linux-x86_64

# define the version of Qt5 that we are going to use. This MUST match the values
# that are expected in the qt_headless.js file
ENV QT_MINOR=11
ENV QT_PATCH=1

# Define the remaining variables that will be needed during the build of the
# Docker image
ENV CMAKE_BUILD_TYPE=Release
ENV GIT_BRANCH=develop
ENV WORKSPACE=/opt/DREAM3D-Dev
ENV SDK_SOURCE_DIR=$DREAM3D_SDK/dream3dsuperbuild
ENV SDK_BUILD_DIR=$SDK_SOURCE_DIR/$CMAKE_BUILD_TYPE
ENV DREAM3D_EXTRA_PLUGINS="ImageProcessing;ITKImageProcessing;MASSIFUtilities;DDDAnalysisToolbox;HEDMAnalysis;TransformationPhase;DREAM3DReview;UCSBUtilities"
ENV BUILD_DIR=$WORKSPACE/DREAM3D-Build/$CMAKE_BUILD_TYPE
ENV SOURCE_DIR=$WORKSPACE/DREAM3D

# Create the DREAM3D_SDK Directory
RUN mkdir -p $DREAM3D_SDK && \
  chmod -R 0777 $DREAM3D_SDK && \
# Install CMake 3.x
  cd $DREAM3D_SDK && \
  wget https://cmake.org/files/v$CMAKE_VERSION/$CMAKE_NAME.tar.gz && \
  tar -xzvf $CMAKE_NAME.tar.gz && \
  rm -rf $CMAKE_NAME.tar.tgz


# Note that the qt_headless.js file has a version of Qt5 specified which MUST
# match this version of Qt
#COPY qt_headless.js $DREAM3D_SDK/qt_headless.js

# Download the correct Qt installer
# RUN cd $DREAM3D_SDK && \
#   wget -c https://download.qt.io/archive/qt/5.$QT_MINOR/5.$QT_MINOR.$QT_PATCH/qt-opensource-linux-x64-5.$QT_MINOR.$QT_PATCH.run

# RUN cd $DREAM3D_SDK && \
#   wget -c http://192.168.242.120/binaries/qt-opensource-linux-x64-5.$QT_MINOR.$QT_PATCH.run && \
#   # Make the download executable
#   cd $DREAM3D_SDK && chmod ugo+rwx qt-opensource-linux-x64-5.$QT_MINOR.$QT_PATCH.run && \
#   # Run the Qt5 installer
#   $DREAM3D_SDK/qt-opensource-linux-x64-5.$QT_MINOR.$QT_PATCH.run --platform minimal --script $DREAM3D_SDK/qt_headless.js && \
#     

# Use CMake to configure the SDK build...
RUN echo "Cloning DREAM3DSuperbuild..." && \
  PATH=$PATH:/opt/qt511/bin && \
  # clone the DREAM3DSuperbuild repo
  cd $DREAM3D_SDK && \
  git clone -b develop https://github.com/bluequartzsoftware/dream3dsuperbuild && \
 # Create a directory for the CMake configuration of the DREAM3D SDK
  mkdir -p $SDK_BUILD_DIR && \
  cd $SDK_BUILD_DIR && \
  $DREAM3D_SDK/$CMAKE_NAME/bin/cmake \
  -DDREAM3D_SDK=$DREAM3D_SDK \
  -DINSTALL_QT5=OFF \
  -DBUILD_TBB=ON \
  -DQMAKE_EXECUTABLE=/opt/qt5${QT_MINOR}/bin/qmake \
  -Dqt5_version_full=5.$QT_MINOR.$QT_PATCH \
  -DQt5_DIR=/opt/qt511/lib/cmake/Qt5 \
  -DBUILD_ITK=ON \
  -DCMAKE_BUILD_TYPE=$CMAKE_BUILD_TYPE \
  $SDK_SOURCE_DIR

  # Build the SDK by compiling sources or downloading prebuilt binaries
  # Be sure to untar the data directories so that the installation will work
RUN cd $SDK_BUILD_DIR && \
  export PATH=$PATH:/opt/qt511/bin && \
  make -j "$(nproc)" && \
  cd $DREAM3D_SDK/DREAM3D_Data/Data && \
  tar -xvzf SmallIN100.tar.gz && \
  tar -xvzf Image.tar.gz

# Create a directory for the actual DREAM.3D source codes
RUN mkdir -p $WORKSPACE && \
  # Clone DREAM.3D, select plugins, and all required repositories
  cd $WORKSPACE && \
# Get dream3d
  git clone -b $GIT_BRANCH https://github.com/BlueQuartzSoftware/DREAM3D && \
# Get required repositories
  git clone -b $GIT_BRANCH https://github.com/BlueQuartzSoftware/CMP && \
  git clone -b $GIT_BRANCH https://github.com/BlueQuartzSoftware/SIMPL && \
  git clone -b $GIT_BRANCH https://github.com/BlueQuartzSoftware/SIMPLView && \
# Get desired plugins
  mkdir -p DREAM3D_Plugins && \
  cd DREAM3D_Plugins && \
# Anisotropy
  git clone -b $GIT_BRANCH https://github.com/dream3d/Anisotropy && \
# MASSIFUtilities
  git clone -b $GIT_BRANCH https://github.com/dream3d/MASSIFUtilities && \
# DDDAnalysisToolbox
  git clone -b $GIT_BRANCH https://github.com/dream3d/DDDAnalysisToolbox && \
# HEDMAnalysis
  git clone -b $GIT_BRANCH https://github.com/dream3d/HEDMAnalysis && \
# TransformationPhase
  git clone -b $GIT_BRANCH https://github.com/dream3d/TransformationPhase && \
# DREAM3DReview
  git clone -b $GIT_BRANCH https://github.com/dream3d/DREAM3DReview && \
# ImageProcessing
  git clone -b $GIT_BRANCH https://github.com/dream3d/ImageProcessing && \
# UCSBUtilities
  git clone -b $GIT_BRANCH https://github.com/dream3d/UCSBUtilities && \
# ITKImageProcessing
  git clone -b $GIT_BRANCH https://github.com/bluequartzsoftware/ITKImageProcessing


RUN mkdir -p $BUILD_DIR && \
  cd $BUILD_DIR && \
  $DREAM3D_SDK/$CMAKE_NAME/bin/cmake \
  -DDREAM3D_SDK=$DREAM3D_SDK \
  -DSIMPLView_BUILD_DOCUMENTATION=OFF \
  -DBUILD_TESTING=OFF \
  -DSIMPL_BUILD_TESTING=OFF \
  -DSIMPL_USE_ITK=ON \
  -DSIMPL_USE_QtWebEngine=OFF \
  -DCMAKE_BUILD_TYPE=$CMAKE_BUILD_TYPE \
  -DDREAM3D_EXTRA_PLUGINS=$DREAM3D_EXTRA_PLUGINS \
  -DCMAKE_INSTALL_PREFIX=$DREAM3D_SDK/dream3d \
  $SOURCE_DIR && \
  # Build DREAM3D
  cd ${BUILD_DIR} && \
  make -j "$(nproc)" && \
  # Create the redistributable package
  make install


## Create a new image
FROM library/ubuntu:18.04

# Install runtime deps
# RUN apt-get update && apt-get -y install \
#   libtbb2 \
#   libqt5core5a \
#   libqt5network5 \

COPY --from=build $DREAM3D_SDK/dream3d/Bin /usr/local/dream3d/Bin

RUN cd /usr/local/dream3d/Bin && \
  rm -r Data && \
# The environment setup
  echo "export PATH=/usr/local/dream3d/Bin:\$PATH" >> ~/setupEnvironment && \
  echo "export LD_LIBRARY_PATH=/usr/local/dream3d/Bin" >> ~/setupEnvironment && \
# The command we will run
  echo "source ~/setupEnvironment" >> ~/runPipelineRunner && \
  echo "PipelineRunner -p \$1" >> ~/runPipelineRunner

ENTRYPOINT ["bash", "/root/runPipelineRunner"]
