FROM debian:stable-slim

# Install Packages (basic tools, cups, basic drivers, HP drivers)
RUN apt-get update \
&& apt-get install -y \
  sudo \
  whois \
  cups \
  cups-client \
  cups-bsd \
  cups-filters \
  foomatic-db-compressed-ppds \
  printer-driver-all \
  smbclient \
  printer-driver-cups-pdf \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

# Install brother driver
RUN ln -s /etc/init.d/cupsys /etc/init.d/lpd && mkdir /tmp/driver
COPY files/brgenml1cupswrapper-3.1.0-1.i386.deb /tmp/driver
COPY files/brgenml1lpr-3.1.0-1.i386.deb /tmp/driver
RUN dpkg -i --force-all /tmp/driver/brgenml1lpr-3.1.0-1.i386.deb && dpkg -i --force-all /tmp/driver/brgenml1cupswrapper-3.1.0-1.i386.deb && dpkg -l | grep Brother | rm /tmp/driver/*.deb

# Add user and disable sudo password checking
RUN useradd \
  --groups=sudo,lp,lpadmin \
  --create-home \
  --home-dir=/home/print \
  --shell=/bin/bash \
  --password=$(mkpasswd print) \
  print \
&& sed -i '/%sudo[[:space:]]/ s/ALL[[:space:]]*$/NOPASSWD:ALL/' /etc/sudoers

# Configure the service's to be reachable
RUN /usr/sbin/cupsd \
  && while [ ! -f /var/run/cups/cupsd.pid ]; do sleep 1; done \
  && cupsctl --remote-admin --remote-any --share-printers \
  && kill $(cat /var/run/cups/cupsd.pid)

# Patch the default configuration file to only enable encryption if requested
RUN sed -e '0,/^</s//DefaultEncryption IfRequested\n&/' -i /etc/cups/cupsd.conf

# Default shell
CMD ["/usr/sbin/cupsd", "-f"]
