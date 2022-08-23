FROM python:3.9-slim-bullseye

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
