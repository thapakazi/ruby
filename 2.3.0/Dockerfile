FROM phusion/baseimage:0.9.19
MAINTAINER CloudFactory <devops@cloudfactory.com>

ENV DEBIAN_FRONTEND noninteractive
ENV RUBY_VERSION 2.3.0

RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confold" && \
    apt-get install -y --fix-missing curl ca-certificates git build-essential libtool autoconf ruby deborphan && \
    apt-get -y clean && apt-get -y autoclean && apt-get -y autoremove && \
    deborphan | xargs apt-get remove --purge -y && \
    rm -rf /var/lib/apt/lists/*

# Install chruby
# ADD https://github.com/postmodern/chruby/archive/v0.3.9.tar.gz /tmp/chruby.tar.gz
RUN curl -SL https://github.com/postmodern/chruby/archive/v0.3.9.tar.gz | tar -xzvC /tmp && \
    cd /tmp/chruby-0.3.9 && make install && \
    cd .. && rm -rf chruby-0.3.9

# Install ruby-install
# ADD https://github.com/postmodern/ruby-install/archive/master.tar.gz /tmp/ruby-install.tar.gz
RUN curl -SL https://github.com/postmodern/ruby-install/archive/master.tar.gz | tar -xzvC /tmp && \
    cd /tmp/ruby-install-master && make install && \
    cd .. && rm -rf ruby-install-master

RUN apt-get update && \
    ruby-install \
    -p https://raw.github.com/skaes/rvm-patchsets/master/patches/ruby/${RUBY_VERSION}/railsexpress/01-skip-broken-tests.patch \
    -p https://raw.github.com/skaes/rvm-patchsets/master/patches/ruby/${RUBY_VERSION}/railsexpress/02-improve-gc-stats.patch \
    -p https://raw.github.com/skaes/rvm-patchsets/master/patches/ruby/${RUBY_VERSION}/railsexpress/03-display-more-detailed-stack-trace.patch \
    ruby ${RUBY_VERSION} -- --disable-install-rdoc --enable-shared CFLAGS="-O3" && \
    rm -rf /usr/local/src /var/lib/apt/lists/*


RUN echo '[ -n "$BASH_VERSION" ] || [ -n "$ZSH_VERSION" ] || return' >> /etc/profile.d/chruby.sh && \
    echo 'source /usr/local/share/chruby/chruby.sh' >> /etc/profile.d/chruby.sh && \
    echo 'chruby ruby' >> /etc/profile.d/default_ruby.sh


# Clean up APT when done.
# RUN apt-get remove --purge -y ruby && apt-get -y autoremove && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
  	BUNDLE_BIN="$GEM_HOME/bin" \
  	BUNDLE_SILENCE_ROOT_WARNING=1 \
  	BUNDLE_APP_CONFIG="$GEM_HOME"
# ENV PATH $BUNDLE_BIN:$PATH
ENV PATH /opt/rubies/ruby-${RUBY_VERSION}/bin:$BUNDLE_BIN:$PATH

RUN  mkdir -p "$GEM_HOME" "$BUNDLE_BIN"&& chmod 777 "$GEM_HOME" "$BUNDLE_BIN" && \
    { \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	  } > ~/.gemrc && \
    chruby-exec ruby -- gem install bundler && \
    apt-get remove --purge -y ruby && \
    apt-get -y autoremove && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# RUN chruby-exec ruby -- gem install bundler
