FROM thehyve/ohdsi-r-base:3.6.1

RUN installGithub.r \
      OHDSI/DataQualityDashboard \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

COPY dqd /usr/bin/

CMD ["/usr/bin/dqd"]
