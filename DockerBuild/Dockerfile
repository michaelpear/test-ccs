FROM ubuntu:16.04 as baseinstall
#Prepare image tools
RUN  dpkg --add-architecture i386  && \
    apt-get update && \
    apt-get -y install curl git ca-certificates python-dev && \
    dpkg --add-architecture i386  && \
    apt-get -y install libc6:i386
WORKDIR /install-ti
#get the file with options to install in Code Composer Studio
COPY docker-response.txt .
ENV TI_INSTALL /opt/ti
ENV CCS_INSTALL $TI_INSTALL/ccsv7
RUN  curl -SL \
 http://software-dl.ti.com/ccs/esd/CCSv7/CCS_7_2_0/exports/CCS7.2.0.00013_web_linux-x64.tar.gz | \
 tar -xz  && \
 ./ccs_setup_linux64_7.2.0.00013.bin --response-file /install-ti/docker-response.txt \
     --mode unattended  --prefix $TI_INSTALL --test true && \
     rm -r $CCS_INSTALL/install_logs $CCS_INSTALL/install_info $CCS_INSTALL/tirex \
     $CCS_INSTALL/uninstallers $CCS_INSTALL/uninstall_ccs $CCS_INSTALL/uninstall_ccs.dat \
     $TI_INSTALL/xdctools_3_32_02_25_core $TI_INSTALL/xdctools_3_50_02_20_core

# build cppcheck and put into location for copy to end system.

FROM alpine as cppcheckinstall

RUN \
	T="$(date +%s)" && \
	apk add --no-cache -t .required_apks git make g++ pcre-dev && \
	mkdir -p /usr/src /src /opt/cppcheck && cd /usr/src && \
	git clone https://github.com/danmar/cppcheck.git && \
	cd cppcheck && \
	git checkout tags/1.80 && \
	make install PREFIX=/opt/cppcheck CFGDIR=/opt/cppcheck/cfg HAVE_RULES=yes CXXFLAGS="-O2 -DNDEBUG --static" -j `getconf _NPROCESSORS_ONLN` && \
	strip /opt/cppcheck/bin/cppcheck && \
	apk del .required_apks && \
	rm -rf /usr/src && \
	T="$(($(date +%s)-T))" && \
	printf "Build time: %dd %02d:%02d:%02d\n" "$((T/86400))" "$((T/3600%24))" "$((T/60%60))" "$((T%60))"

FROM ubuntu:16.04 as finalimage
RUN  dpkg --add-architecture i386  && \
    apt-get update && \
    apt-get -y install curl git ca-certificates python-dev && \
    dpkg --add-architecture i386  && \
    apt-get -y install libc6:i386
WORKDIR /opt/ti
COPY --from=baseinstall /opt/ti .
WORKDIR /opt/cppcheck
#Now get cppcheck 
COPY --from=cppcheckinstall /opt/cppcheck .


