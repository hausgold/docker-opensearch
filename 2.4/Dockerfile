FROM opensearchproject/opensearch:2.4.0
LABEL org.opencontainers.image.authors="containers@hausgold.de"
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

# Copy custom scripts
COPY config/*.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/*

# Configure haproxy
COPY config/haproxy.conf /etc/haproxy/haproxy.cfg

# Configure supervisord
COPY config/supervisor/* /etc/supervisord.d/

# Define the command to run per default
CMD ["/usr/bin/supervisord", "-nc", "/etc/supervisord.conf"]
