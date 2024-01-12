# grafana-mailcow
Dashboard for Mailcow Monitoring with Prometheus and Grafana

# docker-compose.override.yml additions:
* this is asuming the existence of a docker network named "rproxy" in which a reverse proxy listens and secures requests to those exporters

```
networks:
  rproxy:
    name: rproxy
    external: true

services:

  mailcow-exporter:
    image: thej6s/mailcow-exporter
    container_name: mailcow-exporter
    restart: unless-stopped
    depends_on:
      postfix-mailcow:
        condition: service_started
    environment:
      - MAILCOW_EXPORTER_HOST=##REPLACE##
      - MAILCOW_EXPORTER_API_KEY=##REPLACE##
    ports:
      - 9099
    networks:
      - mailcow-network
      - rproxy

  postfix-exporter:
    #image: blackflysolutions/postfix-exporter:latest # amd64 only
    image: boky/postfix-exporter:latest               # supports arm
    container_name: postfix-exporter
    restart: unless-stopped
    depends_on:
      postfix-mailcow:
        condition: service_started
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /data/mailcow/postfix/public/showq:/var/spool/postfix/public/showq:ro
    command: --docker.enable --docker.container.id=mailcowdockerized-postfix-mailcow-1 --web.telemetry-path="/postfix-metrics"
    ports:
      - 9154
    networks:
      - rproxy
```

# example prometheus config:

```
  - job_name: 'mailcow'
    static_configs:
      - targets:
        - 'server1.fqdn'
        labels:
          site: dc1
          type: server
      - targets:
        - 'server2.fqdn'
        labels:
          site: dc2
          type: server

  - job_name: 'mailcow-postfix'
    metrics_path: "/postfix-metrics"
    static_configs:
      - targets:
        - 'server1.fqdn'
        - 'server2.fqdn'
```