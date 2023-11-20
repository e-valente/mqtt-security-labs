FROM jasonish/suricata:latest as builder 

# RUN apk add mosquitto mosquitto-clients iptables
RUN dnf -y install mosquitto iptables
COPY docker-entrypoint.sh /

ENTRYPOINT ["/docker-entrypoint.sh"]