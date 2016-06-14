FROM ubuntu:14.04

MAINTAINER xiang.sean.ma@gmail.com

ENV PYTHONPATH /opt/caffe-segnet/python
ENV PATH $PATH:/opt/caffe-segnet/.build_release/tools

# faster apt source
RUN echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty main restricted universe multiverse \n\
deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-updates main restricted universe multiverse \n\
deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-backports main restricted universe multiverse \n\
deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-security main restricted universe multiverse" > /etc/apt/sources.list

RUN apt-get update && apt-get install -y \
  bc \
  git \
  unzip \
  wget \
  curl \

  # for caffe
  libprotobuf-dev \
  libleveldb-dev \
  libsnappy-dev \
  libopencv-dev \
  libhdf5-serial-dev \
  protobuf-compiler \
  libatlas-base-dev \
  libgflags-dev \
  libgoogle-glog-dev \
  liblmdb-dev \
  libboost-all-dev \

  # for caffe python
  python-dev \
  python-pip \
  python-numpy \
  # for scipy
  gfortran \
  # fix: InsecurePlatformWarning: A true SSLContext object is not available.
  libffi-dev \
  libssl-dev \

  && apt-get clean \
  && rm -rf /var/lib/apt/lists/

RUN cd /opt && git clone https://github.com/alexgkendall/caffe-segnet.git && cd caffe-segnet

WORKDIR /opt/caffe-segnet

# Build Caffe core
RUN cp Makefile.config.example Makefile.config && \
    echo "CPU_ONLY := 1" >> Makefile.config && \
    make -j"$(nproc)" all

# Install python deps
RUN pip install --upgrade pip && \
    # fix: InsecurePlatformWarning: A true SSLContext object is not available.
    pip install pyopenssl ndg-httpsclient pyasn1 && \
    for req in $(cat python/requirements.txt); do pip install $req; done
RUN easy_install -U scikit-image

# Build Caffe python
RUN make -j"$(nproc)" pycaffe

# test + run tests
RUN make -j"$(nproc)" test
# RUN cd /opt/caffe && make runtest

# remove warning for 1394 (maybe related to opencv)
# http://stackoverflow.com/questions/12689304/ctypes-error-libdc1394-error-failed-to-initialize-libdc1394
sudo ln /dev/null /dev/raw1394
