FROM python:3.11-bookworm

# make python3 to default
RUN update-alternatives --install /usr/bin/python python /usr/local/bin/python3.11 10
ENV PYTHONUNBUFFERED 1

# install system dependencies
RUN apt-get -y update \
 && apt-get -y upgrade --with-new-pkgs --autoremove \
 && apt-get -y install \
    bzip2 ca-certificates curl less netbase tini unzip wget \
    apt-transport-https gnupg git software-properties-common \
    direnv libpq-dev \
 && apt-get -y autoclean \
 && apt-get -y autoremove --purge \
 && rm -rf /var/lib/apt/lists/*
 
# Copy local context to `/app` inside container (see .dockerignore)
ARG APP_DIR=/test-litestar
RUN mkdir -p ${APP_DIR}

WORKDIR ${APP_DIR}
COPY . .

# Create virtualenv which will be copied into final container
ENV VIRTUAL_ENV=${APP_DIR}/.venv
ENV PATH="${VIRTUAL_ENV}/bin:$PATH"

RUN python -m venv ${VIRTUAL_ENV}
RUN pip install --upgrade --no-cache-dir pip poetry

RUN poetry env use ${VIRTUAL_ENV}/bin/python \
 && poetry env info \
 && poetry install
 
ENV LITESTAR_APP=test_litestar.app:app
EXPOSE 8000

# entrypoint
ENTRYPOINT ["tini", "-g", "--", "poetry", "run"]
CMD ["litestar", "run", "--host", "0.0.0.0", "--reload"]
