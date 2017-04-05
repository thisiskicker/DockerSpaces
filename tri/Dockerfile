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
RUN touch /etc/ssh/sshd_config && \
echo "Port 6666 \
Protocol 2 \
HostKey /etc/ssh/ssh_host_rsa_key \
HostKey /etc/ssh/ssh_host_dsa_key \
HostKey /etc/ssh/ssh_host_ecdsa_key \
HostKey /etc/ssh/ssh_host_ed25519_key \
UsePrivilegeSeparation yes \
KeyRegenerationInterval 3600 \
ServerKeyBits 1024 \
SyslogFacility AUTH \
LogLevel INFO \
LoginGraceTime 120 \
PermitRootLogin no \
StrictModes yes \
RSAAuthentication yes \
PubkeyAuthentication yes \
IgnoreRhosts yes \
RhostsRSAAuthentication no \
PermitEmptyPasswords no \
ChallengeResponseAuthentication no \
X11Forwarding yes \
X11DisplayOffset 10 \
PrintMotd no \
PrintLastLog yes \
TCPKeepAlive yes \
AcceptEnv LANG LC_* \
Subsystem sftp /usr/lib/openssh/sftp-server \
UsePAM yes" > /etc/ssh/sshd_config && \
service ssh restart

#configure fail2ban to watch ssh on port 6666
RUN echo '[INCLUDES] \
before = paths-debian.conf \
[DEFAULT] \
ignoreip = 127.0.0.1/8 \
ignorecommand = \
bantime  = 600 \
findtime  = 600 \
maxretry = 5 \
backend = auto \
usedns = warn \
logencoding = auto \
enabled = false \
filter = %(__name__)s \
destemail = root@localhost \
sender = root@localhost \
mta = sendmail \
protocol = tcp \
chain = INPUT \
port = 0:65535 \
banaction = iptables-multiport \
action_ = %(banaction)s[name=%(__name__)s, bantime="%(bantime)s", port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"] \
action_mw = %(banaction)s[name=%(__name__)s, bantime="%(bantime)s", port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"] \
            %(mta)s-whois[name=%(__name__)s, dest="%(destemail)s", protocol="%(protocol)s", chain="%(chain)s"] \
action_mwl = %(banaction)s[name=%(__name__)s, bantime="%(bantime)s", port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"] \
             %(mta)s-whois-lines[name=%(__name__)s, dest="%(destemail)s", logpath=%(logpath)s, chain="%(chain)s"] \
action_xarf = %(banaction)s[name=%(__name__)s, bantime="%(bantime)s", port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"] \
             xarf-login-attack[service=%(__name__)s, sender="%(sender)s", logpath=%(logpath)s, port="%(port)s"] \
action_cf_mwl = cloudflare[cfuser="%(cfemail)s", cftoken="%(cfapikey)s" ] \
                %(mta)s-whois-lines[name=%(__name__)s, dest="%(destemail)s", logpath=%(logpath)s, chain="%(chain)s"] \
action_blocklist_de  = blocklist_de[email="%(sender)s", service=%(filter)s, apikey="%(blocklist_de_apikey)s"] \
action_badips = badips.py[category="%(name)s", banaction="%(banaction)s"] \
action = %(action_mwl)s \
[sshd] \
port    = 6666 \
logpath = %(sshd_log)s \
[sshd-ddos] \
port    = ssh \
logpath = %(sshd_log)s \
[dropbear] \
port     = ssh \
logpath  = %(dropbear_log)s \
[selinux-ssh] \
port     = ssh \
logpath  = %(auditd_log)s \
maxretry = 5' > /etc/fail2ban/jail.local && \
systemctl restart fail2ban.service

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
RUN adduser u --quiet --disabled-password --gecos "user"
RUN echo "u:99rubies" | chpasswd

#set up vnc and the vnc service
RUN cat >startVNC.sh <<EOF \
\#!/bin/sh \
vncserver :1 -geometry 1024x768 -depth 16 -pixelformat rgb565 -localhost \
EOF && \
cat >stopVNC.sh <<EOF \
\#!/bin/sh \
vncserver -kill :1 \
EOF && \
chmod +x startVNC.sh && \
chmod +x stopVNC.sh && \
cd /etc/systemd/system && \
cat >vnc.service <<EOF \
[Unit] \
Description=vnc \
After=network.target \
[Service] \
Type=forking \
User=u \
ExecStart=/home/u/startVNC.sh \
ExecStop=/home/u/stopVNC.sh \
Restart=on-abort \
[Install] \
WantedBy=multi-user.target \
EOF && \
chmod +664 vnc.service \
systemctl enable vnc.service && \
screen -d -m -S vncpass && \
screen -S vncpass -p 0 -X stuff "su u^M" && \
screen -S vncpass -p 0 -X stuff "vncpasswd^M" && \
screen -S vncpass -p 0 -X stuff "9rubies^M" && \
screen -S vncpass -p 0 -X stuff "9rubies^M" && \
screen -S vncpass -p 0 -X stuff "y^M" && \
screen -S vncpass -p 0 -X stuff "8rubies^M" && \
screen -S vncpass -p 0 -X stuff "8rubies^M" && \
screen -S vncpass -p 0 -X stuff "exit^M" && \
screen -S vncpass -p 0 -X stuff "exit^M" && \
systemctl restart vnc.service

#tests
CMD java -version && \
node -v && \
sshd -h && \
cowsay "hello" && \
cat /etc/ssh/sshd_config | cowsay
