#label for the type of service
LABEL service = "rsb"

#the image that the container is based off of
FROM ubuntu:16:04

#installs the needed packages below
RUN apt-get update -y && apt-get install -y \
	xorg \
	lxde-core \
	libtasn1-bin \
	libtsan1-3-bin \
	screen \
	libxslt1.1 \
	python-software-properties && \
apt-get remove xscreensaver -y && apt-get autoremove -y && rm -rf /var/lib/apt/lists/*

#add the java repo and installs it
RUN add-apt-repository ppa:webupd8team/java -y && apt-get update \
echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
apt-get install oracle-java8-installer -y \
rm -rf /var/lib/apt/lists/*

#download and install tiger vnc
RUN wget -O tigerVNCserver_1.7.1-ubuntu-amd64.deb https://bintray.com/tigervnc/stable/download_file?file_path=ubuntu-16.04LTS%2Famd64%2Ftigervncserver_1.7.1-1ubuntu1_amd64.deb && dpkg -i tigerVNCserver_1.7.1-ubuntu-amd64.deb

CMD top
