# ---- use a base image to compile requirements / save image size -----
FROM python:3.10-slim as base

WORKDIR /app

# Sets utf-8 encoding for Python
ENV LANG=C.UTF-8
# Turns off writing .pyc files
ENV PYTHONDONTWRITEBYTECODE=1
# Seems to speed things up
ENV PYTHONUNBUFFERED=1

COPY ./requirements.txt  /app/requirements.txt

RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

RUN . /opt/venv/bin/activate && \
    pip install --no-cache-dir -Ur requirements.txt

# ----------------- main container -------------------------

FROM python:3.10-slim

ENV DEBIAN_FRONTEND=noninteractive
ARG ROLE
ENV ROLE=${ROLE}
RUN echo "Building ${ROLE} container."
RUN if [ "$ROLE" = "flask" ]; then \
    apt update -y && apt install -yq postgis; \
    elif [ "$ROLE" = "celery" ]; then \
    apt update -y && apt install -yq postgis libimage-exiftool-perl ffmpeg; \
    fi
RUN apt clean
RUN apt autoremove

WORKDIR /app

# Allows specifying a custom UID and GID during build
ARG USER_ID
ENV USER_ID=${USER_ID:-1000}
ARG GROUP_ID
ENV GROUP_ID=${GROUP_ID:-1000}

RUN groupadd bayanat --gid=$GROUP_ID
RUN useradd bayanat --uid $USER_ID --gid=$GROUP_ID

# Sets utf-8 encoding for Python
ENV LANG=C.UTF-8
# Turns off writing .pyc files
ENV PYTHONDONTWRITEBYTECODE=1
# Seems to speed things up
ENV PYTHONUNBUFFERED=1

COPY . /app
# copy compiled virtualenv
COPY --from=base /opt/venv /opt/venv

# environments are passed directly to the image
ARG ENV_FILE
COPY ${ENV_FILE:-.env} /app

COPY --chown=bayanat:bayanat ./flask/bin/entrypoint.sh /usr/local/bin/entrypoint.sh

RUN chmod 550 /usr/local/bin/entrypoint.sh

ENV PATH="/opt/venv/bin:$PATH"

USER bayanat

CMD ["/usr/local/bin/entrypoint.sh"]