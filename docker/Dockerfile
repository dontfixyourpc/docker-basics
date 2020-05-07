# Name: localghost 
# Description: Creates a promtail image from scratch
# Date: 05/07/2020


FROM ubuntu:18.04

RUN apt-get update && \
  apt-get install -qy \
  tzdata ca-certificates libsystemd-dev && \
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
ENV PROM_VERSION=/v1.4.1/promtail-linux-amd64.zip
ENV PROM_URL=https://github.com/grafana/loki/releases/download/${PROM_VERSION}
RUN apt-get update && apt-get -y install curl unzip && curl -L ${PROM_URL} -o promtail-linux-amd64.zip && unzip promtail-linux-amd64.zip \
  && chmod 750 promtail-linux-amd64 && mv /promtail-linux-amd64 /usr/bin/promtail
EXPOSE 9080
COPY promtail-docker-config.yaml /etc/promtail/docker-config.yaml
CMD ["-config.file=/etc/promtail/docker-config.yaml"]
ENTRYPOINT ["/usr/bin/promtail"]