# Closely following https://www.mercurial-scm.org/wiki/HgWebDirStepByStep

FROM ubuntu:20.04
LABEL maintainer="Matthäus G. Chajdas <dev@anteru.net>"

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -qq update
RUN apt-get -y install apache2 libapache2-mod-wsgi-py3 python3-pip

# This will get us the latest mercurial, while apt-get install mercurial will
# end up installing some old version
RUN pip3 install mercurial

RUN mkdir /etc/apache2/hg

COPY hg/hgweb.config    /var/hg/
COPY hg/hgwebdir.wsgi   /var/hg/

# We completely overwrite the Apache config here
# If you edit main.conf/000-default.conf, make sure
# to check how the unmodified one looks first
COPY apache/hg.conf /etc/apache2/hg/main.conf
COPY apache/000-default.conf /etc/apache2/sites-enabled/000-default.conf

COPY hgrc /etc/mercurial/hgrc

EXPOSE 80

RUN chown -R www-data:www-data /var/hg
CMD service apache2 restart && /bin/bash
