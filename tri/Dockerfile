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
	wget \
	openssh-server \
	fail2ban \
	cowsay && \
apt-get remove xscreensaver -y && \
apt-get autoclean && apt-get autoremove --purge -y && \
rm -rf /var/lib/apt/lists/* /var/cache/apt/* /tmp/* /var/tmp/*

#download and install tiger vnc
RUN wget -O tigerVNCserver_1.7.1-ubuntu-amd64.deb https://bintray.com/tigervnc/stable/download_file?file_path=ubuntu-16.04LTS%2Famd64%2Ftigervncserver_1.7.1-1ubuntu1_amd64.deb && dpkg -i tigerVNCserver_1.7.1-ubuntu-amd64.deb && \
rm tigerVNCserver_1.7.1-ubuntu-amd64.deb

#install node v7
RUN curl -sL https://deb.nodesource.com/setup_7.x | bash - && \
apt-get install nodejs -y && \
apt-get autoclean && apt-get autoremove --purge -y && \
rm -rf /var/lib/apt/lists/* /var/cache/apt/* /tmp/* /var/tmp/*

#install node libraries
RUN npm install chalk system-sleep robotjs

#change ssh server config to make it more secure (makes ssh run on port 6666, no root login, & ssh key login)
ADD /config/sshd_config /etc/ssh/
RUN service ssh restart

#configure fail2ban to watch ssh on port 6666
ADD /config/jail.local /etc/fail2ban/
RUN service fail2ban restart

#harden the network by preventing souuce routing of incoming packets and log malformed IP's && prevent ip spoofing
RUN echo "net.ipv4.conf.all.rp_filter = 1 \
net.ipv4.conf.default.rp_filter = 1 \
net.ipv4.icmp_echo_ignore_broadcasts = 1 \
net.ipv4.conf.all.accept_source_route = 0 \
net.ipv6.conf.all.accept_source_route = 0 \
net.ipv4.conf.default.accept_source_route = 0 \
net.ipv6.conf.default.accept_source_route = 0 \
net.ipv4.conf.all.send_redirects = 0 \
net.ipv4.conf.default.send_redirects = 0 \
net.ipv4.tcp_syncookies = 1 \
net.ipv4.tcp_max_syn_backlog = 2048 \
net.ipv4.tcp_synack_retries = 2 \
net.ipv4.tcp_syn_retries = 5 \
net.ipv4.conf.all.log_martians = 1 \
net.ipv4.icmp_ignore_bogus_error_responses = 1 \
net.ipv4.conf.all.accept_redirects = 0 \
net.ipv6.conf.all.accept_redirects = 0 \
net.ipv4.conf.default.accept_redirects = 0 \
net.ipv6.conf.default.accept_redirects = 0 \
net.ipv4.icmp_echo_ignore_all = 1" > /etc/sysctl.conf
RUN echo "order bind, hosts \
nospoof on" > /etc/host.conf 

#create the user u
RUN adduser u --quiet --disabled-password --gecos "user" && \
echo "u:99rubies" | chpasswd && \
usermod -aG sudo u

#set up vnc and the vnc service
ADD /script/*VNC.sh /home/u/
ADD /config/vnc.service /etc/systemd/system/
RUN chmod +x /home/u/*VNC.sh && \
chmod +664 /etc/systemd/system/vnc.service && \
systemctl enable vnc.service
ADD /config/passwd /home/u/.vnc/
RUN chown u /home/u/.vnc/passwd
RUN systemctl start vnc.service

#tests
CMD java -version && \
node -v && \
sshd -h && \
cowsay "hello" && \
sat /etc/ssh/sshd_config | cowsay
