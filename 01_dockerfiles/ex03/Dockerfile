FROM debian:jessie

# Install required packages
RUN apt-get update -q \ 
	&& DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
	lsb-release \
	ca-certificates \
	openssh-server \
	wget \
	apt-transport-https \
	vim \
	nano

RUN echo "deb https://packages.gitlab.com/gitlab/gitlab-ce/debian/ `lsb_release -cs` main" > /etc/apt/sources.list.d/gitlab_gitlab-ce.list
RUN wget -q -O - https://packages.gitlab.com/gpg.key | apt-key add -
RUN apt-get update && apt-get install -yq --no-install-recommends gitlab-ce

# Manage SSHD through runit
RUN mkdir -p /opt/gitlab/sv/sshd/supervise \
    && mkfifo /opt/gitlab/sv/sshd/supervise/ok \
	&& printf "#!/bin/sh\nexec 2>&1\numask 077\nexec /usr/sbin/sshd -D" > /opt/gitlab/sv/sshd/run \
	&& chmod a+x /opt/gitlab/sv/sshd/run \
	&& ln -s /opt/gitlab/sv/sshd /opt/gitlab/service \
	&& mkdir -p /var/run/sshd

# Disabling use DNS in ssh since it tends to slow connecting
RUN echo "UseDNS no" >> /etc/ssh/sshd_config

# Prepare default configuration
RUN ( \
	echo "" && \
	echo "# Docker options" && \
	echo "# Prevent Postgres from trying to allocate 25% of total memory" && \
	echo "postgresql['shared_buffers'] = '1MB'" ) >> /etc/gitlab/gitlab.rb && \
	mkdir -p /assets/ && \
	cp /etc/gitlab/gitlab.rb /assets/gitlab.rb

EXPOSE 443 80 22

RUN ( \
	echo 'external_url "https://192.168.99.100"' && \
	echo "gitlab_rails['gitlab_shell_ssh_port'] = 4242" && \
	echo "nginx['redirect_http_to_https'] = true" && \
	echo "nginx['ssl_certificate'] = '/etc/gitlab/ssl/192.168.99.100.crt'" &&  \
	echo "nginx['ssl_certificate_key'] = '/etc/gitlab/ssl/192.168.99.100.key'") >> /etc/gitlab/gitlab.rb && \
	mkdir -p /etc/gitlab/ssl

VOLUME ["/etc/gitlab", "/var/opt/gitlab", "/var/log/gitlab"]

COPY cert.key /etc/gitlab/ssl/192.168.99.100.key
COPY cert.crt /etc/gitlab/ssl/192.168.99.100.crt
COPY wrapper .

RUN chmod 777 ./wrapper
CMD ["./wrapper"]

#how to build ?
#docker build -t ex03img /path/to/Dockerfile

#how to run
#docker run --privileged --name gitlab -p 80:80 -p 443:443 -p 4242:22 -d  ex03img:latest

#attention a ce que les ip a la fin du dockerfile, correspondent a l ip de Char
#generer clef SSH
#rentrer la clef public sur gitlab
#pour changer le type de push : git remote add *name *Git ssh or https link 
