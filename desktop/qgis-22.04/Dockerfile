FROM harbor.cyverse.org/vice/xpra/qgis:22.04

USER root

RUN export DEBIAN_FRONTEND=noninteractive

# Install PPA for various CloudCompare dependencies

RUN apt-get update && apt-get upgrade -y --allow-unauthenticated && \
    apt-get update && apt-get install -y --allow-unauthenticated build-essential software-properties-common apt-utils && \
    apt-get update && \
    add-apt-repository --yes ppa:ubuntu-x-swat/updates && \
    apt-get update

# Install various dependencies
RUN apt-get update && apt-get install -y --allow-unauthenticated \
    clang \
    cmake \
    curl \
    ffmpeg \
    gcc \
    kmod \
    libboost-all-dev \
    qtbase5-dev \
    libqt5opengl5-dev \
    libqt5svg5-dev \
    libcgal-qt5-dev \
    libeigen3-dev \
    libfreenect-dev \
    libtbb-dev \
    libxerces-c-dev \
    libxpm-dev \
    libzmq3-dev \
    pkg-config \
    qttools5-dev \
    rsync \
    openjdk-8-jdk \
    openjdk-8-jre-headless && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN ln -s /usr/lib/x86_64-linux-gnu/libvtkCommonCore-6.2.so /usr/lib/libvtkproj4.so

# Install OpenGL Drivers
RUN apt-get update && \
    apt-get install -y --allow-unauthenticated freeglut3-dev mesa-common-dev mesa-utils
    
# Compile LASZIP
RUN cd /opt && git clone https://github.com/LASzip/LASzip && \
    cd LASzip && \
    git tag && \ 
    git checkout tags/3.4.3 && \
    mkdir build && \
    cd build && \
    cmake .. && \
    #./configure && \
    make -j48 && \
    make -j48 install

# Install PCL
RUN apt-get update && apt-get install libflann-dev libpcl-dev -y

# CC suggestion
RUN /sbin/ldconfig -v

# Ready to install and compile CloudCompare
ENV LD_LIBRARY_PATH /opt/qt512/lib/
RUN apt-get update
RUN cd /opt && git clone --recursive https://github.com/cloudcompare/CloudCompare.git CloudCompare && \
    cd /opt/CloudCompare && git checkout tags/v2.12.4 && \
    cd /opt/CloudCompare/ && mkdir build && \
    cd build && \
    # set release and compiler type
    cmake -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CXX_COMPILER=clang++ \
    # set path for QT5
    -DQT5_ROOT_PATH=/usr/share/qt5 \	    
    -DEIGEN_ROOT_DIR=/usr/local/include/eigen3/ \
    -DCOMPILE_CC_CORE_LIB_WITH_TBB=ON \
   # set plugins
    -DINSTALL_EXAMPLE_PLUGIN=ON \
    -DINSTALL_EXAMPLE_GL_PLUGIN=ON \
    -DINSTALL_EXAMPLE_IO_PLUGIN=ON \
    -DINSTALL_QADDITIONAL_IO_PLUGIN=ON \
    -DINSTALL_QANIMATION_PLUGIN=ON \
    -DINSTALL_QBROOM_PLUGIN=ON \
    -DINSTALL_QCOMPASS_PLUGIN=ON \
    -DINSTALL_QCSF_PLUGIN=ON \
    -DINSTALL_QEDL_PLUGIN=ON \
    -DINSTALL_QFACETS_PLUGIN=ON \
    -DINSTALL_QHOUGH_NORMALS_PLUGIN=ON \
    -DINSTALL_QHPR_PLUGIN=ON \
    -DINSTALL_QM3C2_PLUGIN=ON \
    -DINSTALL_QPCV_PLUGIN=ON \
    # agisoft photoscan (metashape)
    -DINSTALL_QPHOTOSCAN_IO_PLUGIN=ON \
    # poisson recon
    -DINSTALL_QPOISSON_RECON_PLUGIN=ON \
    -DPOISSON_RECON_WITH_OPEN_MP=ON \
    # sra
    -DINSTALL_QSRA_PLUGIN=ON \
    -DINSTALL_QSSAO_PLUGIN=ON \
    # ffmpeg
    -DWITH_FFMPEG_SUPPORT=ON \
    -DFFMPEG_INCLUDE_DIR=/usr/include/x86_64-linux-gnu \
    -DFFMPEG_LIBRARY_DIR=/usr/lib/x86_64-linux-gnu \
    # gdal
    -DOPTION_USE_GDAL=ON \
    # pdal
    -DPLUGIN_IO_QPDAL=TRUE \
    -DOPTION_PDAL_LAS=ON \
    # json
    -DJSON_ROOT_DIR=/usr/include/jsoncpp \
    -DOPTION_USE_DXF_LIB=ON \
    -DOPTION_USE_SHAPE_LIB=ON \
    -DOPTION_USE_DXF_LIB=ON \
    # pcl
    -DINSTALL_QPCL_PLUGIN=ON \
    # build with CGAL - getting an error 2021-05-27
    -DCCCORELIB_USE_CGAL=ON \
    #-DCOMPILE_CC_CORE_LIB_WITH_CGAL=ON \
    .. && \
    make -j48 && \
    make -j48 install

ENV LD_LIBRARY_PATH /usr/local/bin   

# Agisoft Metashape
RUN cd /opt && wget https://s3-eu-west-1.amazonaws.com/download.agisoft.com/metashape-pro_2_0_2_amd64.tar.gz && \
    tar xvf metashape-pro_2_0_2_amd64.tar.gz && \
    cd /opt/metashape-pro && wget https://s3-eu-west-1.amazonaws.com/download.agisoft.com/Metashape-2.0.2-cp37.cp38.cp39.cp310.cp311-abi3-linux_x86_64.whl && \
    wget https://s3-eu-west-1.amazonaws.com/download.agisoft.com/metashape-java-api-2.0.2.zip && unzip *.zip

# Create Desktop Icons

COPY cc.desktop /usr/share/applications/
COPY metashape.desktop /usr/share/applications/

RUN cd /opt && rm -rf *.tar.bz2 && \
    rm -rf *.tar.gz && rm -rf *.deb

# fix QGIS startup errors in VICE related to libQT5
RUN strip --remove-section=.note.ABI-tag /usr/lib/x86_64-linux-gnu/libQt5Core.so.5

USER user

# fix Python path problem with Conda and QGIS
ENV PATH /usr/lib/xpra:/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/VirtualGL/bin:/usr/local/go/bin:/usr/lib/grass78/bin:/usr/lib/grass78/scripts

VOLUME /tmp/.X11-unix

CMD xpra start :10 --video-encoders=nvenc --bind-tcp=0.0.0.0:9876 --html=on --start-child=xterm --exit-with-children=no --daemon=no --xvfb="/usr/bin/Xvfb +extension Composite -screen 0 1920x1080x24+32 -nolisten tcp -noreset" --pulseaudio=no --notifications=no --bell=no 

