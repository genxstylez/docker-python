FROM alpine:edge
MAINTAINER Sam Liu <genxstylez@gmail.com>

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV             LANG C.UTF-8
ENV             PYTHON_VERSION 2.7.11
ENV             PYTHONUNBUFFERED True
ENV             PIP_NO_CACHE_DIR off
ENV             PIP_DISABLE_PIP_VERSION_CHECK on

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV             PYTHON_PIP_VERSION 8.1.2


RUN             apk --no-cache add \
                    python=$PYTHON_VERSION-r3 \
                    curl && \

                python -m ensurepip --default-pip && \
                curl -fSL 'https://bootstrap.pypa.io/get-pip.py' | python2 && \
                pip install --no-cache-dir --upgrade pip==$PYTHON_PIP_VERSION


RUN             apk --no-cache add --virtual .builddeps \
                    build-base \
                    python-dev=$PYTHON_VERSION-r3 \
                    openssl-dev \
                    libffi-dev \
                    zlib-dev  \
                    jpeg-dev \
                    libpng-dev && \
                ln -s /lib/libz.so /usr/lib/. && \
                pip install wheel==0.29.0 && \
                pip install \
                    cython==0.23.4 \
                    pyOpenSSL==0.15.1 \
                    ndg-httpsclient==0.4.0 \
                    pyasn1==0.1.9 \
                    pyasn1-modules==0.0.8 \
                    pycparser==2.14 \
                    cryptography==1.2.3 \
                    cffi==1.5.2 && \
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
                pip uninstall -y cython && \
                apk del .builddeps && \
                find /usr/ \
                    \( -type d -a -name test -o -name tests \) \
                    -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
                    -exec rm -rf '{}' + && \
                find . -type f -name "__pycache__" -exec rm -rf '{}' +

ONBUILD COPY    . /usr/src/app
