FROM nvidia/cuda-ppc64le:9.1-cudnn7-devel-ubuntu16.04
MAINTAINER Yasuhiro Nakajima <e35294@jp.ibm.com>

# install base
RUN apt-get update && DEBIAN_FRONTEND=noninteractive  apt-get install -y \
    wget curl git apt-transport-https apt-utils vim htop tree zip unzip \
    openssh-server iputils-ping net-tools language-pack-ja-base language-pack-ja fonts-ipafont-mincho build-essential openjdk-8-jdk cmake && \
    apt-get clean

# install python3 module
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    python python3 python3-pip python3-numpy python3-scipy python3-matplotlib python3-dev python3-wheel python3-setuptools libgfortran3 && \
    apt-get clean

# upgrade module
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    apt-get clean

#setup python3
RUN update-alternatives --install /usr/bin/python python /usr/bin/python2 10
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 11

# 1. ------------ Build the MXNet core shared library ------------------ 
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    libopenblas-dev liblapack-dev libopencv-dev libcurl4-openssl-dev libgtest-dev libatlas-base-dev python-opencv && \
    cd / && \
# Download MXNet sources and build MXNet core shared library
    git clone --recursive -b 1.1.0 https://github.com/apache/incubator-mxnet.git mxnet && \
    cd mxnet && \
    make -j $(nproc) USE_OPENCV=1 USE_BLAS=openblas USE_CUDA=1 USE_CUDA_PATH=/usr/local/cuda USE_CUDNN=1 && \
    rm -r build && \
    
# 2. ------------ Build the MXNet Python binding ------------------ 
   
   cd python && \
   pip3 install --upgrade pip && \
   pip3 install -e . && \
   apt-get clean && \

ENV PYTHONPATH=/mxnet/python

# 3. ------------ Build and install the nccl  ------------------
   
   cd / && \
   git clone https://github.com/NVIDIA/nccl.git && \
   cd /nccl  && \
   make CUDA_HOME=/usr/local/cuda-9.0 test && \
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/nccl/build/lib

#     ------------------------ END --------------------------------

# setup ssh
RUN mkdir /var/run/sshd
RUN echo root:$(echo "passw0rd") | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i '/^AcceptEnv/s/^/#/' /etc/ssh/sshd_config
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
RUN echo "export PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:$PATH" >> /root/.bashrc && \
    echo "export LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64" >> /root/.bashrc


# setup timezone and locale
ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN echo "export LANG=ja_JP.UTF-8" >> /root/.bashrc
RUN echo "export LANGUAGE=ja_JP.UTF-8" >> /root/.bashrc
ENV LANGUAGE=ja_JP
ENV LANG=ja_JP.UTF-8

# import python modules
RUN pip3 install IPython==5.3
RUN pip3 install jupyter
RUN mkdir /root/.jupyter
RUN echo "c.NotebookApp.ip = '*'" > ~/.jupyter/jupyter_notebook_config.py


CMD []
