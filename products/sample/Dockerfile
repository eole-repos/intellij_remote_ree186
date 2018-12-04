# Create a base image available for each container

FROM amazonlinux:2018.03

# Copy necessary files

COPY products/sample/ruby-enterprise-1.8.6-20090610.tar.gz \
     products/sample/ree_openssl_no_ec2m.patch \
     products/sample/no_sslv2.diff \
     products/sample/openssl-1.0.patch \
     products/sample/setup_8.x \
     products/sample/daemontools-0.76.tar.gz \
     products/sample/daemontools-0.76.errno.patch \
     products/sample/get-pip.py \
     /tmp/

WORKDIR /tmp

# Set timezone
RUN /bin/cp -f /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

# Install necessary packages
RUN yum update -y \
 && yum reinstall -y glibc-common \
 && yum install -y git svn mysql-client redis-tools nano vi vim \
    gcc-c++ glibc-headers openssl-devel readline libyaml-devel readline-devel \
    zlib zlib-devel libffi-devel libxml2 libxslt libxml2-devel libxslt-devel sqlite-devel bzip2 \
    patch file-devel mysql-devel php php-pear php-devel php-pecl-imagick autoconf \
    ImageMagick ImageMagick-devel ImageMagick-perl \
 && : "Install pip, setuptools" \
 && python get-pip.py

# locale設定
ENV LANG ja_JP.UTF-8

# Install rbenv and ruby-build
ENV RBENV_ROOT /opt/rbenv
ENV PATH $RBENV_ROOT/bin:$PATH
ENV PATH $RBENV_ROOT/shims:$PATH
RUN git clone git://github.com/sstephenson/rbenv.git /opt/rbenv \
 && git clone https://github.com/sstephenson/ruby-build.git $RBENV_ROOT/plugins/ruby-build

# Install Grunt
RUN /bin/bash setup_8.x && yum install -y nodejs && npm install -g grunt

# Install Ruby
RUN : "Version ree-1.8.6-2009.06" \
 && tar xvzf ruby-enterprise-1.8.6-20090610.tar.gz \
 && cd ruby-enterprise-1.8.6-20090610 \
 && patch -d source/ -p1 < /tmp/openssl-1.0.patch \
 && patch -d source/ -p1 < /tmp/no_sslv2.diff \
 && ./installer --no-tcmalloc --auto $RBENV_ROOT/versions/ree-1.8.6-2009.06 --dont-install-useful-gems \
 && : "Version ree-1.8.7-2012.02" \
 && cat /tmp/ree_openssl_no_ec2m.patch | \
    CFLAGS="-O2 -fno-tree-dce -fno-optimize-sibling-calls" \
    CONFIGURE_OPTS="--no-tcmalloc" \
    rbenv install --patch ree-1.8.7-2012.02 \
 && : "Version 2.0.0-p648" \
 && rbenv install 2.0.0-p648 \
 && : "Version 2.1.2p95" \
 && rbenv install 2.1.2 \
 && : "Version 2.3.0p0" \
 && rbenv install 2.3.0

# Install daemontools
RUN cd /tmp && tar xvzf daemontools-0.76.tar.gz \
 && cd admin/daemontools-0.76 \
 && patch -p1 < /tmp/daemontools-0.76.errno.patch \
 && ./package/compile \
 && cd command \
 && sed -i 's/command/usr\/local\/bin/g' svscanboot \
 && install -o root -g root -m 755 -s multilog /usr/local/bin \
 && install -o root -g root -m 755 -s pgrphack /usr/local/bin \
 && install -o root -g root -m 755 -s readproctitle /usr/local/bin \
 && install -o root -g root -m 755 -s setlock /usr/local/bin \
 && install -o root -g root -m 755 -s setuidgid /usr/local/bin \
 && install -o root -g root -m 755 -s softlimit /usr/local/bin \
 && install -o root -g root -m 755 -s supervise /usr/local/bin \
 && install -o root -g root -m 755 -s svc /usr/local/bin \
 && install -o root -g root -m 755 -s svok /usr/local/bin \
 && install -o root -g root -m 755 -s svscan /usr/local/bin \
 && install -o root -g root -m 755 -s svstat /usr/local/bin \
 && install -o root -g root -m 755 -s tai64n /usr/local/bin \
 && install -o root -g root -m 755 -s tai64nlocal /usr/local/bin \
 && install -o root -g root -m 755 svscanboot /usr/local/bin

# Create a product container

EXPOSE 3000
WORKDIR /srv/www/rails/sample

# Copy necessary files
COPY ./volumes/sample/Gemfile /srv/www/rails/sample

RUN : "Set Ruby version" \
 && rbenv global ree-1.8.6-2009.06 \
 && : "Install gems" \
 && gem install rubygems-update -v 1.3.7 --no-rdoc --no-ri \
 && rbenv exec update_rubygems \
 && gem install bundler -v 1.0.21 --no-rdoc --no-ri \
 && rbenv rehash \
 && bundle install

#CMD script/server -p 3000
