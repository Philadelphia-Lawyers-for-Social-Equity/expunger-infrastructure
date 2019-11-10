FROM dan0mau/devuan-ascii

ENV APPDIR /srv/plse

ARG EXPUNGER_USER
ARG EXPUNGER_PASS
ARG EXPUNGER_KEY
ARG DJANGO_SETTINGS_MODULE

# System initialization

WORKDIR $APPDIR
RUN apt-get update -y -qq && \
    apt-get upgrade -y -qq && \
    apt-get install -y -qq \
    apache2 \
    apache2-dev \
    build-essential \
    git \
    python3 \
    python3-pip \
    python3-dev \
    python3-setuptools \
    libcurl4-openssl-dev \
    libssl-dev \
    tzdata \
    libffi-dev \
    libpq-dev \
    vim \
    # Cleanup
    && apt-get autoremove -y \
    && apt-get purge -y \
    && apt-get clean -y

# timezone-related fixes
RUN ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime && \
    dpkg-reconfigure --frontend noninteractive tzdata

# Provide a local user environment
# https://jtreminio.com/blog/running-docker-containers-as-current-host-user/#ok-so-what-actually-works
ARG USER_ID
ARG GROUP_ID

RUN if [ ${USER_ID:-0} -ne 0 ] && [ ${GROUP_ID:-0} -ne 0 ];\
    then \
        if getent group ${EXPUNGER_USER}; then groupdel ${EXPUNGER_USER}; fi; \
        if getent passwd ${EXPUNGER_USER}; then userdel -f ${EXPUNGER_USER}; fi; \
        groupadd -g ${GROUP_ID} ${EXPUNGER_USER} && \
        useradd -m -l -u ${USER_ID} -g ${EXPUNGER_USER} ${EXPUNGER_USER} \
    ; else \
        if ! getent group ${EXPUNGER_USER}; then groupadd ${EXPUNGER_USER}; fi; \
        if ! getent passwd ${EXPUNGER_USER}; then useradd -m -l -g ${EXPUNGER_USER} ${EXPUNGER_USER}; fi \
    ; fi

RUN echo ${EXPUNGER_USER}:${EXPUNGER_PASS} | chpasswd

RUN mkdir -p ${APPDIR}/static
RUN chown --changes --silent --no-dereference --recursive ${EXPUNGER_USER}:${EXPUNGER_USER} ${APPDIR}

# App install
USER ${EXPUNGER_USER}
COPY platform/src/ .
ENV PATH "$PATH:/home/${EXPUNGER_USER}/.local/bin"
RUN pip3 install --user -r requirements.txt
RUN pip3 install --user mod_wsgi
RUN python3 ./manage.py collectstatic --noinput

ENTRYPOINT python3 ./manage.py makemigrations && python3 ./manage.py migrate && python3 ./manage.py runmodwsgi
