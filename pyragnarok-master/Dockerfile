FROM continuumio/miniconda3:latest

COPY pyragnarok/ pyragnarok/
WORKDIR .

COPY requirements.txt .
COPY scripts/start.sh  start.sh

RUN chmod +x start.sh

RUN apt-get update && apt-get install -y \
    build-essential zip && pip install numpy && \
    pip install -r requirements.txt

EXPOSE 5000
ENV CONFIG_FILE=/config/pyragnarok.ini

CMD ["./start.sh"]
