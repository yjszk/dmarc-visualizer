FROM python:3.9-alpine3.16

RUN apk add --update --no-cache libxml2-dev libxslt-dev
RUN apk add --update --no-cache --virtual .build_deps build-base libffi-dev \
    && pip install parsedmarc msgraph-core==0.2.2

COPY parsedmarc.ini /
