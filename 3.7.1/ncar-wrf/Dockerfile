FROM centos:latest
MAINTAINER John Exby <exby@ucar.edu>
# 
# This Dockerfile compiles WRF from source during "docker build" step
ENV WRF_VERSION 3.7.1
#
RUN yum -y update
RUN yum -y install file gcc gcc-gfortran gcc-c++ glibc.i686 libgcc.i686 libpng-devel jasper jasper-devel hostname m4 make perl tar tcsh time wget which zlib zlib-devel epel-release
#
# now get 3rd party EPEL builds of netcdf and openmpi dependencies
RUN yum -y install netcdf-openmpi-devel.x86_64 netcdf-fortran-openmpi-devel.x86_64 netcdf-fortran-openmpi.x86_64 hdf5-openmpi.x86_64 openmpi.x86_64 openmpi-devel.x86_64
#
WORKDIR /wrf
#
# Download original sources
#
RUN curl -SL http://www2.mmm.ucar.edu/wrf/src/WRFV$WRF_VERSION.TAR.gz | tar zxC /wrf \
 && curl -SL http://www2.mmm.ucar.edu/wrf/src/WPSV$WRF_VERSION.TAR.gz | tar zxC /wrf
#
# Set environment for interactive container shells
#
RUN echo export LDFLAGS="-lm" >> /etc/bashrc \
 && echo export NETCDF=/wrf/netcdf_links >> /etc/bashrc \
 && echo export JASPERINC=/usr/include/jasper/ >> /etc/bashrc \
 && echo export JASPERLIB=/usr/lib64/ >> /etc/bashrc \
 && echo export LD_LIBRARY_PATH="/usr/lib64/openmpi/lib" >> /etc/bashrc \
 && echo export PATH="/usr/lib64/openmpi/bin:$PATH" >> /etc/bashrc \
 && echo setenv LDFLAGS "-lm" >> /etc/csh.cshrc \
 && echo setenv NETCDF "/wrf/netcdf_links" >> /etc/csh.cshrc \
 && echo setenv JASPERINC "/usr/include/jasper/" >> /etc/csh.cshrc \
 && echo setenv JASPERLIB "/usr/lib64/" >> /etc/csh.cshrc \
 && echo setenv LD_LIBRARY_PATH "/usr/lib64/openmpi/lib" >> /etc/csh.cshrc \
 && echo setenv PATH "/usr/lib64/openmpi/bin:$PATH" >> /etc/csh.cshrc
#
# Build WRF first
# 
# input 34 and 1 to configure script alternative line = && echo -e "34\r1\r" | ./configure
RUN mkdir netcdf_links \
 && ln -sf /usr/include/openmpi-x86_64/ netcdf_links/include \
 && ln -sf /usr/lib64/openmpi/lib netcdf_links/lib \
 && export NETCDF=/wrf/netcdf_links \
 && export JASPERINC=/usr/include/jasper/ \
 && export JASPERLIB=/usr/lib64/ \
 && cd ./WRFV3 \
 && ./configure <<< $'34\r1\r' \
 && sed -i -e '/^DM_CC/ s/$/ -DMPI2_SUPPORT/' ./configure.wrf \
 && /bin/csh ./compile em_real
#
# Build WPS second
#
# input 1 to configure script
RUN cd ./WPS \
 && export NETCDF=/wrf/netcdf_links \
 && export JASPERINC=/usr/include/jasper/ \
 && export JASPERLIB=/usr/lib64/ \
 && ./configure <<< $'1\r' \
 && sed -i -e 's/-L$(NETCDF)\/lib/-L$(NETCDF)\/lib -lnetcdff /' ./configure.wps \
 && /bin/csh ./compile
#
ENV LD_LIBRARY_PATH /usr/lib64/openmpi/lib
ENV PATH  /usr/lib64/openmpi/bin:$PATH
#
# copy in a couple custom scripts
COPY run-wrf /wrf
COPY docker-clean /wrf
RUN chmod +x /wrf/run-wrf \
 && chmod +x /wrf/docker-clean
CMD ["/wrf/run-wrf"]
