FROM opensearchproject/opensearch:1.3.6
MAINTAINER Hermann Mayer "hermann.mayer@hausgold.de"
# See: https://bit.ly/3gbAmcR

# You can change this environment variable on run's with -e
ENV MDNS_HOSTNAME=opensearch.local

# Keep the settings low for default
ENV OPENSEARCH_JAVA_OPTS='-Xms96m -Xmx96m'
ENV discovery.type='single-node'
ENV DISABLE_INSTALL_DEMO_CONFIG='true'
ENV DISABLE_SECURITY_PLUGIN='true'
# ENV compatibility.override_main_response_version='true'

# We run our configurations as root user from now on
USER root

# Install system packages
RUN amazon-linux-extras install epel -y && \
  yum install -y \
    dbus avahi avahi-tools nss-mdns haproxy supervisor

# Copy avahi.sh
COPY config/avahi.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/avahi.sh

# Configure haproxy
COPY config/haproxy.conf /etc/haproxy/haproxy.cfg

# Configure supervisord
COPY config/supervisor/* /etc/supervisord.d/

# Define the command to run per default
CMD /usr/bin/supervisord -nc /etc/supervisord.conf
