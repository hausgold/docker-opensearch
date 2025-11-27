FROM opensearchproject/opensearch:3.2.0
LABEL org.opencontainers.image.authors="containers@hausgold.de"
# See: https://bit.ly/4lnAi6s

# You can change this environment variable on run's with -e
ENV MDNS_HOSTNAME=opensearch.local

# Keep the settings low for default
ENV OPENSEARCH_JAVA_OPTS='-Xms256m -Xmx256m'
ENV discovery.type='single-node'
ENV cluster.routing.allocation.disk.threshold_enabled='false'
ENV DISABLE_INSTALL_DEMO_CONFIG='true'
ENV DISABLE_SECURITY_PLUGIN='true'

# We run our configurations as root user from now on
USER root

# Install system packages
RUN EPEL='https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/Packages'; \
  dnf install -y \
    dbus-daemon avahi avahi-tools haproxy  \
    "${EPEL}/n/nss-mdns-0.15.1-3.1.el9.x86_64.rpm" \
    "${EPEL}/s/supervisor-4.2.5-6.el9.noarch.rpm"

# Copy custom scripts
COPY config/*.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/*

# Configure haproxy
COPY config/haproxy.conf /etc/haproxy/haproxy.cfg

# Configure supervisord
COPY config/supervisor/* /etc/supervisord.d/

# Define the command to run per default
CMD ["/usr/bin/supervisord", "-nc", "/etc/supervisord.conf"]
