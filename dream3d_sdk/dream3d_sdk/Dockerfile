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
  python3-distutils \
  python3-pip \
  ninja-build

# Load up the mkdocs python modules
RUN /usr/bin/pip3 install mkdocs-material

# Define the DREAM3D_SDK directory
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

# Create the DREAM3D_SDK Directory
RUN mkdir -p $DREAM3D_SDK && \
  chmod -R 0777 $DREAM3D_SDK && \
# Install CMake 3.x
  cd $DREAM3D_SDK && \
  wget https://cmake.org/files/v$CMAKE_VERSION/$CMAKE_NAME.tar.gz && \
  tar -xzvf $CMAKE_NAME.tar.gz && \
  rm -rf $CMAKE_NAME.tar.tgz

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
  tar -xvzf Image.tar.gz && \
# Remove the entire DREAM3D_SDK Build directory
  rm $DREAM3D_SDK/cmake-$CMAKE_VERSION.$CMAKE_PATCH-Linux-x86_64.tar.gz && \
  rm -rf $DREAM3D_SDK/superbuild/boost && \
  rm -rf $DREAM3D_SDK/superbuild/CMP && \
  rm -rf $DREAM3D_SDK/superbuild/discount && \
  rm -rf $DREAM3D_SDK/superbuild/DREAM3D_Data && \
  rm -rf $DREAM3D_SDK/superbuild/Eigen && \
  rm -rf $DREAM3D_SDK/superbuild/haru && \
  rm -rf $DREAM3D_SDK/superbuild/hdf5 && \
  rm -rf $DREAM3D_SDK/superbuild/pybind11 && \
  rm -rf $DREAM3D_SDK/superbuild/qwt && \
  rm -rf $DREAM3D_SDK/superbuild/tbb


# Create a shell script that will kick off a build
RUN echo "#!/bin/bash" > build.sh && \
  echo "set -e" >> build.sh && \
  echo "$DREAM3D_SDK/cmake-$CMAKE_VERSION.$CMAKE_PATCH-Linux-x86_64/bin/ctest -VV -S /dream3d/Support/circleci/CircleContinuous.cmake" >> build.sh && \
  chmod u+x build.sh

ENTRYPOINT ["/build.sh"]

ARG BUILD_DATE
ARG IMAGE=dream3d/dream3d:dream3d_sdk
ARG VCS_REF
ARG VCS_URL
LABEL org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name=$IMAGE \
      org.label-schema.description="Image containing DREAM.3D environment to build and test DREAM3D" \
      org.label-schema.url="https://github.com/bluequartzsoftware/dream3d" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url=$VCS_URL \
      org.label-schema.schema-version="1.0"

# Remove the entire DREAM3D_SDK Build directory
#RUN echo "DREAM3D_SDK: $DREAM3D_SDK" && \
#  echo "DREAM3DSuperbuild: $SDK_BUILD_DIR" && \
#  ls -laF $DREAM3D_SDK/

#RUN more  $DREAM3D_SDK/DREAM3D_SDK.cmake


