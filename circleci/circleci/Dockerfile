FROM dream3d/dream3d:dream3d_sdk AS build
LABEL Author="BlueQuartz Software <info@bluequartz.net>"
MAINTAINER BlueQuartz Software <dream3d@bluequartz.net>

SHELL ["/bin/bash", "-c"]


# Define the DREAM3D_SDK director y
ENV DREAM3D_SDK=/opt/DREAM3D_SDK

# Define the version of CMake that we will be using
ENV CMAKE_VERSION=3.13
ENV CMAKE_PATCH=2
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
ENV BUILD_DIR=/DREAM3D-Build/$CMAKE_BUILD_TYPE
ENV SOURCE_DIR=/dream3d


# Create a directory for the actual DREAM.3D source codes
# Clone DREAM.3D, select plugins, and all required repositories
RUN cd / && \
# Get dream3d
  git clone -b $GIT_BRANCH https://github.com/BlueQuartzSoftware/dream3d && \
# Get required repositories
  git clone -b $GIT_BRANCH https://github.com/BlueQuartzSoftware/CMP && \
  git clone -b $GIT_BRANCH https://github.com/BlueQuartzSoftware/SIMPL && \
  git clone -b $GIT_BRANCH https://github.com/BlueQuartzSoftware/SIMPLView

# List out all of the DREAM3D Extra Plugins that we want to build
ENV DREAM3D_EXTRA_PLUGINS="Anisotropy;MASSIFUtilities;DDDAnalysisToolbox;HEDMAnalysis;TransformationPhase;DREAM3DReview;ImageProcessing;UCSBUtilities;ITKImageProcessing"


RUN cd / && \
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


RUN cd / && \
  mkdir -p $BUILD_DIR && \
  cd $BUILD_DIR && \
  $DREAM3D_SDK/$CMAKE_NAME/bin/cmake \
  -DDREAM3D_SDK=$DREAM3D_SDK \
  -DSIMPLView_BUILD_DOCUMENTATION=OFF \
  -DBUILD_TESTING=ON \
  -DSIMPL_BUILD_TESTING=ON \
  -DSIMPL_USE_ITK=ON \
  -DSIMPL_USE_QtWebEngine=OFF \
  -DCMAKE_BUILD_TYPE=$CMAKE_BUILD_TYPE \
  -DDREAM3D_EXTRA_PLUGINS=$DREAM3D_EXTRA_PLUGINS \
  -DCMAKE_INSTALL_PREFIX=/dream3d-install \
  $SOURCE_DIR && \
  # Build DREAM3D
  cd ${BUILD_DIR} && \
  make -j "$(nproc)" && \
  # Create the redistributable package
  make install
