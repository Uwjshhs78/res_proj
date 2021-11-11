# Using CentOS 6 base image and VNC
# Version 1

# Pull the rhel image from the local repository
FROM centos:6.6
MAINTAINER Ken Evensen

USER root
RUN yum install yum install -y tigervnc-server; yum clean all; rm -rf /var/cache/yum
RUN yum install -y xorg-x11-server-utils xterm gnome-session; yum clean all; rm -rf /var/cache/yum
RUN /bin/dbus-uuidgen > /var/lib/dbus/machine-id
RUN useradd basicuser; mkdir -p /home/basicuser/.vnc
ADD xstartup /home/basicuser/.vnc/
ADD passwd /home/basicuser/.vnc/
RUN chown -R basicuser:basicuser /home/basicuser; chmod a+x /home/basicuser/.vnc/xstartup; chmod 600 /home/basicuser/.vnc/passwd
EXPOSE 5901

ENTRYPOINT ["/usr/bin/vncserver", "-fg"]
