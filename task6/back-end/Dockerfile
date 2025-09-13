FROM ubuntu:latest

RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y python3 python3-pip \
    && rm -rf /var/lib/apt/lists/*

RUN rm /usr/lib/python*/EXTERNALLY-MANAGED || true
RUN pip3 install flask flask-cors

WORKDIR /app
COPY ./api.py /app/api.py

CMD ["python3", "/app/api.py"]


