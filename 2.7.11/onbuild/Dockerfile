FROM alpine:3.3
MAINTAINER Sam Liu <genxstylez@gmail.com>

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV             LANG C.UTF-8
ENV             PYTHON_VERSION 2.7.11
ENV             PYTHON_PIP_VERSION 7.1.2
ENV             PYTHONUNBUFFERED True
ENV             PIP_NO_CACHE_DIR off
ENV             PIP_DISABLE_PIP_VERSION_CHECK on

# gpg: key 18ADD4FF: public key "Benjamin Peterson <benjamin@python.org>" imported
ENV GPG_KEY C01E1CAD5EA2C4F0B8E3571504C367C218ADD4FF

ENV PYTHON_VERSION 2.7.11

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 8.0.3

RUN set -ex \
    && apk add --no-cache --virtual .fetch-deps curl gnupg \
    && curl -fSL "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" -o python.tar.xz \
    && curl -fSL "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" -o python.tar.xz.asc \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
    && gpg --batch --verify python.tar.xz.asc python.tar.xz \
    && rm -r "$GNUPGHOME" python.tar.xz.asc \
    && mkdir -p /usr/src \
    && tar -xJC /usr/src -f python.tar.xz \
    && mv "/usr/src/Python-$PYTHON_VERSION" /usr/src/python \
    && rm python.tar.xz \
    \
    && apk add --no-cache --virtual .build-deps  \
        bzip2-dev \
        gcc \
        libc-dev \
        linux-headers \
        make \
        ncurses-dev \
        openssl-dev \
        pax-utils \
        readline-dev \
        sqlite-dev \
        zlib-dev \
    && cd /usr/src/python \
    && ./configure --enable-shared --enable-unicode=ucs4 \
    && make -j$(getconf _NPROCESSORS_ONLN) \
    && make install \
    && curl -fSL 'https://bootstrap.pypa.io/get-pip.py' | python2 \
    && pip install --no-cache-dir --upgrade pip==$PYTHON_PIP_VERSION \
    && find /usr/local \
        \( -type d -a -name test -o -name tests \) \
        -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
        -exec rm -rf '{}' + \
    && runDeps="$( \
        scanelf --needed --nobanner --recursive /usr/local \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
    )" \
    && apk add --virtual .python-rundeps $runDeps \
    && apk del .build-deps .fetch-deps \
    && rm -rf /usr/src/python \
    && mkdir -p /usr/src/app/

WORKDIR /usr/src/app/
CMD ["python"]

ONBUILD COPY    requirements.txt /usr/src/app/
ONBUILD RUN     cat requirements.txt | grep -oh -E "builddeps:(\w|,|-)*" | awk '{split($0,a,":"); print a[2]}' | awk '{gsub(/,/, "\n", $0); print $0}' | sort -u | xargs apk --no-cache add --virtual .builddeps && \
                cat requirements.txt | grep -oh -E "rundeps:(\w|,|-)*" | awk '{split($0,a,":"); print a[2]}' | awk '{gsub(/,/, "\n", $0); print $0}' | sort -u | xargs apk --no-cache add --virtual .rundeps && \
                pip install -r requirements.txt && \
                apk del .builddeps && \
                find /usr/ \
                    \( -type d -a -name test -o -name tests \) \
                    -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
                    -exec rm -rf '{}' + && \
                find . -type f -name "__pycache__" -exec rm -rf '{}' +
ONBUILD COPY    . /usr/src/app