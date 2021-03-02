FROM thehyve/ohdsi-r-base:3.6.3

ARG GITHUB_PAT

RUN installGithub.r \
    OHDSI/ROhdsiWebApi \
    EHDEN/CatalogueExport \
    EHDEN/CdmInspection \
    OHDSI/DataQualityDashboard \
  && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

COPY dqd /usr/bin/
