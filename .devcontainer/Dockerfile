# development image
FROM python:3.9-slim-bullseye as dev

# --------------------------------------------------------------------------------------------------
# add a non-root user (see https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user)
ARG USERNAME=flask
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user and install additional requirements
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    # also add git for development & clean up
    && apt-get install -y git \
    && apt-get clean
# --------------------------------------------------------------------------------------------------

# set the work dir in the image to /app
WORKDIR /app

# install requirements
RUN pip install --no-cache-dir flask

# copy files from local system to /app
# COPY ./app ./

# set path to flask app and enable development mode
ENV FLASK_APP=app.py
ENV FLASK_DEBUG=1

# start flask when running containers based on this image
ENTRYPOINT [ "flask", "run" ]
CMD ["--host", "0.0.0.0", "--port", "5000"]

# production image
# (we don't need the user and the additional stuff, so simply start from the base image again)
FROM python:3.9-slim-bullseye as prod

WORKDIR /app
RUN pip install --no-cache-dir flask waitress
COPY ./app ./

ENTRYPOINT [ "waitress-serve"]
CMD ["--host", "0.0.0.0", "--port", "5000", "app:app"]
