FROM nvidia/cuda:11.0-devel-ubuntu20.04

RUN apt-get update && \
    DEBIAN_FRONTEND="noninteractive" apt-get install -y software-properties-common apt-transport-https

RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && \
    add-apt-repository 'deb [arch=amd64] https://cran.rstudio.com/bin/linux/ubuntu focal-cran40/' && \
    apt-get update && \
    apt-get install -y r-base

RUN apt-get install -y git wget libcurl4-openssl-dev default-jdk-headless libssl-dev libxml2-dev cmake

RUN R CMD javareconf

ENV MAKE="make -j$(nproc)"

RUN R -e 'install.packages(c("ROCR","data.table","R6","devtools","RCurl","jsonlite"), repos = "https://cran.rstudio.com/")'


ARG CACHE_DATE

RUN R -e 'install.packages("h2o", repos = "http://h2o-release.s3.amazonaws.com/h2o/latest_stable_R")'

RUN git clone --recursive https://github.com/dmlc/xgboost && \
    cd xgboost && \
    mkdir build && cd build && cmake .. -DUSE_CUDA=ON -DR_LIB=ON && \
    make install -j

RUN apt-get install -y libboost-dev libboost-system-dev libboost-filesystem-dev ocl-icd-opencl-dev opencl-headers clinfo
RUN mkdir -p /etc/OpenCL/vendors && \
    echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd   ## otherwise lightgm segfaults at runtime (compiles fine without it)
RUN git clone --recursive https://github.com/microsoft/LightGBM && \
    cd LightGBM && sed -i "s/use_gpu <- FALSE/use_gpu <- TRUE/"  R-package/src/install.libs.R && Rscript build_r.R


CMD ["R","--vanilla"]
