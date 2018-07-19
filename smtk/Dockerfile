FROM library/ubuntu:18.04 AS build
MAINTAINER Kitware <kitware@kitware.com>

SHELL ["/bin/bash", "-c"]

RUN apt-get update && apt-get -y upgrade && \
  apt-get -y install \
  git \
  wget \
  build-essential \
  libxt-dev \
  mesa-common-dev \
  freeglut3-dev \
  m4 \
# In case we want to install python packages with pip
  libssl-dev

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

RUN mkdir -p src && \
  pushd . && \
  cd src && \
  git clone --recursive https://gitlab.kitware.com/cmb/cmb-superbuild && \
  cd cmb-superbuild && \
  git checkout b45fd52e && \
  popd && \
  mkdir -p build/smtk && \
  cd build/smtk && \
  cmake \
    -DENABLE_cmb=OFF \
    -DENABLE_opencv=OFF \
    -DENABLE_qt5=OFF \
    -DENABLE_vtkonly=ON \
    -Dsmtk_SOURCE_SELECTION=git \
    -Dvtk_extra_cmake_args="-DModule_vtkIOXdmf2:BOOL=ON" \
    -Dsmtk_extra_cmake_args="-DSMTK_ENABLE_TESTING:BOOL=OFF;-DBUILD_TESTING:BOOL=OFF" \
    -DBUILD_TESTING=OFF \
    ../../src/cmb-superbuild && \
  make -j "$(nproc)"

# Install any pip dependencies.
RUN /build/smtk/install/bin/pip install jinja2

# Put our afrl automation scripts in
RUN cd /usr/local && \
  git clone -b speed https://bitbucket.org/psavery/afrl-automation && \
  rm -rf afrl-automation/inputs

# Create a new image
FROM library/ubuntu:18.04

# Install runtime deps
RUN apt-get update && apt-get -y install \
  freeglut3 \
  libxt6 \
  libssl1.1

ENV smtkRoot /usr/local/smtk

COPY --from=build /build/smtk/install $smtkRoot

COPY --from=build /usr/local/afrl-automation /usr/local/afrl-automation

# Setup environment variables. We must run '. ~/setupEnvironment' for commands
# from here on out.
RUN echo "export PATH=$smtkRoot/bin:\$PATH" >> ~/setupEnvironment && \
    echo "export PYTHONPATH=$smtkRoot/lib/python2.7/site-packages/vtkmodules" \
          >> ~/setupEnvironment && \
    echo "export LD_LIBRARY_PATH=$smtkRoot/lib" >> ~/setupEnvironment && \
    # In case we want to just use bash...
    echo ". ~/setupEnvironment" >> ~/.bashrc
