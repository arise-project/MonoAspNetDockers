FROM ubuntu:16.04

MAINTAINER Jo Shields <jo.shields@xamarin.com>

#based on dockerfile by Michael Friis <friism@gmail.com>

ENV MONO_VERSION 4.6.2.7
ENV LIBGDI_TAG 4.2
ENV XSP_TAG 4.2
ENV MONO_SRV mono-default
ENV WEBSITE WebAdmin

RUN rm /bin/sh && ln -s /bin/bash /bin/sh

RUN apt-get update \
  && apt-get install -y curl python-software-properties build-essential git autoconf libtool automake gettext nginx libglib2.0-dev libjpeg-dev libpng12-dev libgif-dev libexif-dev libx11-dev libxrender-dev libfreetype6-dev libfontconfig1-dev libcairo2-dev libtool-bin git automake gettext make cmake nginx build-essential\
  && rm -rf /var/lib/apt/lists/*

RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF

RUN echo "deb http://download.mono-project.com/repo/debian wheezy/snapshots/$MONO_VERSION main" > /etc/apt/sources.list.d/mono-xamarin.list \
  && apt-get update \
  && apt-get install -y binutils mono-devel ca-certificates-mono fsharp mono-vbnc nuget referenceassemblies-pcl \
  && rm -rf /var/lib/apt/lists/* /tmp/*

WORKDIR /opt
RUN git clone https://github.com/mono/libgdiplus.git 
WORKDIR /opt/libgdiplus
RUN git checkout $LIBGDI_TAG
RUN ./autogen.sh && ./configure && make && make install

WORKDIR /opt
RUN git clone https://github.com/mono/xsp.git
WORKDIR /opt/xsp
RUN git checkout $XSP_TAG
RUN ./autogen.sh --disable-docs && make && make install

RUN mkdir -p /usr/local/etc/mono/fcgi
RUN mkdir -p /usr/local/etc/mono/fcgi/apps-available
RUN mkdir -p /usr/local/etc/mono/fcgi/apps-enabled

RUN echo "# mono config" | tee -a /etc/nginx/fastcgi_params
RUN echo "fastcgi_param  PATH_INFO          \"\";" | tee -a /etc/nginx/fastcgi_params
RUN echo "fastcgi_param  SCRIPT_FILENAME    \$document_root\$fastcgi_script_name;" | tee -a /etc/nginx/fastcgi_params

# new mono default server
RUN echo "server {" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "         listen   80;" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "         server_name  localhost;" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "         location / {" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "                 root `pwd`/www/;" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "                 index index.html index.htm default.aspx Default.aspx;" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "                 fastcgi_index Default.aspx;" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "                 fastcgi_pass 0.0.0.0:9001;" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "                 include /etc/nginx/fastcgi_params;" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "         }" | tee -a /etc/nginx/sites-available/$MONO_SRV
RUN echo "}" | tee -a /etc/nginx/sites-available/$MONO_SRV
# enable it
RUN ln -s /etc/nginx/sites-available/$MONO_SRV /etc/nginx/sites-enabled/$MONO_SRV
# restart nginx
RUN /etc/init.d/nginx restart

WORKDIR /opt
RUN git clone https://epirogov:12345@bitbucket.org/epirogov/test.git
WORKDIR /opt/test
RUN xbuild test.sln

#monoserve servce script /etc/init.d/monoserve
RUN echo "#!/bin/sh" | tee -a /etc/init.d/monoserve
RUN echo "ATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin" | tee -a /etc/init.d/monoserve
RUN echo "DAEMON=/usr/bin/mono" | tee -a /etc/init.d/monoserve
RUN echo "NAME=monoserve" | tee -a /etc/init.d/monoserve
RUN echo "DESC=monoserve" | tee -a /etc/init.d/monoserve

RUN echo "PROGRAM=fastcgi-mono-server4" | tee -a /etc/init.d/monoserve
RUN echo "ADDRESS=0.0.0.0" | tee -a /etc/init.d/monoserve
RUN echo "PORT=9001" | tee -a /etc/init.d/monoserve


RUN echo "MONOSERVER=\$(which \${PROGRAM})" | tee -a /etc/init.d/monoserve
RUN echo "MONOSERVER_PID=\"\"" | tee -a /etc/init.d/monoserve

#todo use $WEBSITE in apps
RUN echo "WEBAPPS=\"/:/opt/sentrex/web/WebAdmin/\"" | tee -a /etc/init.d/monoserve


RUN echo "get_pid() {" | tee -a /etc/init.d/monoserve
RUN echo "	MONOSERVER_PID=\$(ps auxf | grep \${PROGRAM}.exe | grep -v grep | awk '{print \$2}')" | tee -a /etc/init.d/monoserve
RUN echo "}" | tee -a /etc/init.d/monoserve

RUN echo "start_up(){" | tee -a /etc/init.d/monoserve
RUN echo "    get_pid" | tee -a /etc/init.d/monoserve
RUN echo "    if [ -z \"\${MONOSERVER_PID}\" ]; then" | tee -a /etc/init.d/monoserve
RUN echo "	echo \"Starting Mono FastCGI Server...\"" | tee -a /etc/init.d/monoserve
RUN echo "        \${MONOSERVER} /applications=\${WEBAPPS} /socket=tcp:\${PORT} &" | tee -a /etc/init.d/monoserve
RUN echo "       echo \"Mono FastCGI Server (\${PROGRAM}.exe) started correctly on \${ADDRESS}:\${PORT}\"" | tee -a /etc/init.d/monoserve
RUN echo "    else" | tee -a /etc/init.d/monoserve
RUN echo "        echo \"Mono FastCGI Server is already running - PID \${MONOSERVER_PID}\"" | tee -a /etc/init.d/monoserve
RUN echo "    fi" | tee -a /etc/init.d/monoserve
RUN echo "}" | tee -a /etc/init.d/monoserve

RUN echo "shut_down() {" | tee -a /etc/init.d/monoserve
RUN echo "    get_pid" | tee -a /etc/init.d/monoserve
RUN echo "   if [ -n \"\${MONOSERVER_PID}\" ]; then" | tee -a /etc/init.d/monoserve
RUN echo "        kill \${MONOSERVER_PID}" | tee -a /etc/init.d/monoserve
RUN echo "        echo \"Mono FastCGI Server stopped\"" | tee -a /etc/init.d/monoserve
RUN echo "    else" | tee -a /etc/init.d/monoserve
RUN echo "        echo \"Mono FastCGI Server is not running\"" | tee -a /etc/init.d/monoserve
RUN echo "    fi" | tee -a /etc/init.d/monoserve
RUN echo "}" | tee -a /etc/init.d/monoserve

RUN echo "case \"\$1\" in" | tee -a /etc/init.d/monoserve
RUN echo "    start)" | tee -a /etc/init.d/monoserve
RUN echo "        start_up" | tee -a /etc/init.d/monoserve
RUN echo "    ;;" | tee -a /etc/init.d/monoserve
RUN echo "    stop)" | tee -a /etc/init.d/monoserve
RUN echo "        shut_down" | tee -a /etc/init.d/monoserve
RUN echo "    ;;" | tee -a /etc/init.d/monoserve
RUN echo "    restart|force-reload)" | tee -a /etc/init.d/monoserve
RUN echo "        shut_down" | tee -a /etc/init.d/monoserve
RUN echo "        start_up" | tee -a /etc/init.d/monoserve
RUN echo "    ;;" | tee -a /etc/init.d/monoserve
RUN echo "    status)" | tee -a /etc/init.d/monoserve
RUN echo "        get_pid" | tee -a /etc/init.d/monoserve
RUN echo "        if [ -z \"\${MONOSERVER_PID}\" ]; then" | tee -a /etc/init.d/monoserve
RUN echo "            echo \"Mono FastCGI Server is not running\"" | tee -a /etc/init.d/monoserve
RUN echo "        else" | tee -a /etc/init.d/monoserve
RUN echo "            echo \"Mono FastCGI Server is running - PID \${MONOSERVER_PID}\"" | tee -a /etc/init.d/monoserve
RUN echo "        fi" | tee -a /etc/init.d/monoserve
RUN echo "    ;;" | tee -a /etc/init.d/monoserve
RUN echo "    *)" | tee -a /etc/init.d/monoserve
RUN echo "        echo \"Usage: monoserve (start|stop|restart|force-reload|status)\"" | tee -a /etc/init.d/monoserve
RUN echo ";;esac;exit 0" | tee -a /etc/init.d/monoserve
#RUN echo "esac" | tee -a /etc/init.d/monoserv
#RUN echo "exit 0" | tee -a /etc/init.d/monoserv

RUN chmod +x /etc/init.d/monoserve
RUN update-rc.d monoserve defaults
RUN cat /etc/init.d/monoserve 
#RUN /etc/init.d/monoserve start
RUN service monoserve start

RUN service nginx status
RUN service monoserve status
#databse connection string

#RUN curl http://0.0.0.0

EXPOSE 9001