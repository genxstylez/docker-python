FROM alpine:3.3
MAINTAINER Sam Liu <genxstylez@gmail.com>

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV             LANG C.UTF-8
ENV             PYTHON_VERSION 2.7.11
ENV             PYTHONUNBUFFERED True
ENV             PIP_NO_CACHE_DIR off
ENV             PIP_DISABLE_PIP_VERSION_CHECK on

# gpg: key 18ADD4FF: public key "Benjamin Peterson <benjamin@python.org>" imported
ENV GPG_KEY C01E1CAD5EA2C4F0B8E3571504C367C218ADD4FF

ENV PYTHON_VERSION 2.7.11

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 8.0.3


RUN             apk --no-cache add \
                    python=$PYTHON_VERSION-r3 \
                    curl && \

                python -m ensurepip --default-pip && \
                curl -fSL 'https://bootstrap.pypa.io/get-pip.py' | python2 && \
                pip install --no-cache-dir --upgrade pip==$PYTHON_PIP_VERSION


RUN             apk --no-cache add --virtual .builddeps \
                    build-base \
                    python-dev=$PYTHON_VERSION-r3 \
                    openssl-dev=1.0.2g-r0 \
                    libffi-dev=3.2.1-r2 \
                    zlib-dev  \
                    jpeg-dev \
                    libpng-dev && \
                ln -s /lib/libz.so /usr/lib/. && \
                pip install wheel==0.29.0 && \
                apk --no-cache add --virtual .rundeps \
                    zlib \
                    jpeg \
                    libpng && \
                mkdir -p /usr/src/app/

WORKDIR         /usr/src/app/
CMD             ["python"]

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
