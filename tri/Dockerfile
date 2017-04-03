#the image that the container is based off of
FROM isuper/java-oracle:jre_8

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8
ENV DEBIAN_FRONTEND noninteractive

#installs the needed packages below
RUN apt-get update -y && apt-get install -y \
	xorg \
	lxde-core \
	libtasn1-bin \
	libtasn1-3-bin \
	screen \
	libxslt1.1 \
	python-software-properties \
	wget && \
apt-get remove xscreensaver -y && \
apt-get autoclean && apt-get autoremove --purge -y && \
rm -rf /var/lib/apt/lists/* /var/cache/apt/* /tmp/* /var/tmp/*

#download and install tiger vnc
RUN wget -O tigerVNCserver_1.7.1-ubuntu-amd64.deb https://bintray.com/tigervnc/stable/download_file?file_path=ubuntu-16.04LTS%2Famd64%2Ftigervncserver_1.7.1-1ubuntu1_amd64.deb && dpkg -i tigerVNCserver_1.7.1-ubuntu-amd64.deb && \
rm tigerVNCserver_1.7.1-ubuntu-amd64.deb

CMD java -version
