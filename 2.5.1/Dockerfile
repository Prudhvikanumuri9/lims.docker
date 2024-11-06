# Use Python development image instead of slim
FROM python:2.7

# Set one or more individual labels
LABEL maintainer="Prabhatech"
LABEL email="your.email@prabhatech.com"
LABEL senaite.core.version="v2.5.1"

# Set environment variables
ENV PLONE_MAJOR=5.2 \
    PLONE_VERSION=5.2.14 \
    PLONE_MD5=e8e1f774f069026319be3038631e0734 \
    PLONE_UNIFIED_INSTALLER=Plone-5.2.14-UnifiedInstaller-1.0 \
    SENAITE_HOME=/home/senaite \
    SENAITE_USER=senaite \
    SENAITE_INSTANCE_HOME=/home/senaite/senaitelims \
    SENAITE_DATA=/data \
    SENAITE_FILESTORAGE=/data/filestorage \
    SENAITE_BLOBSTORAGE=/data/blobstorage \
    DEBIAN_FRONTEND=noninteractive

# Create the senaite user
RUN useradd --system -m -d $SENAITE_HOME -U -u 500 $SENAITE_USER

# Create directories
RUN mkdir -p $SENAITE_INSTANCE_HOME $SENAITE_FILESTORAGE $SENAITE_BLOBSTORAGE

# Copy Buildout
COPY requirements.txt buildout.cfg $SENAITE_INSTANCE_HOME/

# Copy the scripts first and set permissions
COPY docker-entrypoint.sh /
COPY docker-initialize.py /
COPY build_deps.txt run_deps.txt /

# Make sure scripts are executable and have correct line endings
RUN chmod +x /docker-entrypoint.sh && \
    sed -i 's/\r//g' /docker-entrypoint.sh

# Install runtime dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git \
        gosu \
        libcairo2 \
        libgdk-pixbuf2.0-0 \
        libjpeg62 \
        libopenjp2-7 \
        libpango-1.0-0 \
        libpangocairo-1.0-0 \
        libtiff5 \
        libxml2 \
        libxslt1.1 \
        lynx \
        rsync \
        vim \
    && rm -rf /var/lib/apt/lists/*

# Fetch and set up Plone
RUN wget -O Plone.tgz https://launchpad.net/plone/$PLONE_MAJOR/$PLONE_VERSION/+download/$PLONE_UNIFIED_INSTALLER.tgz \
    && echo "$PLONE_MD5 Plone.tgz" | md5sum -c - \
    && tar -xzf /Plone.tgz \
    && cp -rv $PLONE_UNIFIED_INSTALLER/base_skeleton/* $SENAITE_INSTANCE_HOME \
    && cp -v $PLONE_UNIFIED_INSTALLER/buildout_templates/buildout.cfg $SENAITE_INSTANCE_HOME/buildout-base.cfg \
    && rm -rf $PLONE_UNIFIED_INSTALLER Plone.tgz

# Set up SENAITE
RUN cd $SENAITE_INSTANCE_HOME \
    && pip install -r requirements.txt \
    && buildout \
    && ln -s $SENAITE_FILESTORAGE/ var/filestorage \
    && ln -s $SENAITE_BLOBSTORAGE/ var/blobstorage \
    && chown -R senaite:senaite $SENAITE_HOME $SENAITE_DATA

# Change working directory
WORKDIR $SENAITE_INSTANCE_HOME

# Mount external volume
VOLUME /data

# Expose instance port
EXPOSE 8080

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["start"]