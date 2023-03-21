# PPS
SDR-ADALM-PLUTO

![SDR](/img/pluto-sdr.jpg)

**Informe SRD_PLUTO**

Caracteristicas de la SDR:

 Para conectarnos a la **sdr** utilizamos Secure Shell (ssh) por ejemplo para conectarnos a una en particular **ssh -l root 192.168.1.x** conectados a la vpn en el host. Una vez conectado a la **sdr** en el directorio **/opt/vfat.img** se encuentra informacion relevante a cerca de la arquitectura de la **sdr** (i.e el firmware y el compilador gcc disponible), se puede copiar este archivo a nuestro host de la siguiente manera **scp  root@192.168.1.x:/opt/vfat.img  info.img**. Luego de haber obtenido la informatacion se procede a preparar un contenedor en docker con las mismas especificaciones de firmware y compilador (contexto en nuestro host). De esta manera se podrà realizar aplicaciones en el host para luego ser ejecutadas en el **sdr**.
 
Dockerfile:
    
    FROM ubuntu:latest

    RUN apt update
    RUN apt-get -y upgrade

    RUN apt-get -y install git
    RUN apt-get -y install wget
    RUN apt update && apt install -y make
    RUN apt-get -y install vim
    RUN apt-get -y install iputils-ping
    RUN apt-get -y install unzip
    RUN apt-get -y install xz-utils
    RUN apt-get -y update

    RUN wget -P/usr/local/bin "https://developer.arm.com/-/media/Files/downloads/gnu-a/8.2-2018.08/gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf.tar.xz?revision=51f3ba22-a569-4dda-aedc-7988690c3c17&ln=en&hash=2DC0BEB0D6E413AEC39F456D213578D2"
    RUN cd /usr/local/bin && tar -Jxvf gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf.tar.xz\?revision\=51f3ba22-a569-4dda-aedc-7988690c3c17\&ln\=en\&hash\=2DC0BEB0D6E413AEC39F456D213578D2


    RUN wget -P/home "https://github.com/analogdevicesinc/plutosdr-fw/releases/download/v0.32/sysroot-v0.32.tar.gz"
    RUN cd /home && tar -xf sysroot-v0.32.tar.gz
    RUN cd /home && mv staging $HOME/pluto-0.32.sysroot
    RUN mkdir /tmp/plutoapp
    RUN cd /tmp/plutoapp && wget "https://raw.githubusercontent.com/analogdevicesinc/libiio/master/examples/ad9361-iiostream.c"
    ENV PATH=$PATH:/usr/local/bin/gcc-arm-8.2-2018.08-x86_64-arm-linux-gnueabihf/bin
 
Construccion del contenedor mediante:
    
    sudo docker build -t ubuntu_sdr -f Dockerfile .
Construccion de la imagen mediante:

    sudo docker run -i -t --net=host ubuntu_sdr


*--net=host* : Permite que la imagen este en la misma red que el host.
