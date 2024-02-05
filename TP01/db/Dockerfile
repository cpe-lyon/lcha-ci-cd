FROM postgres:14.1-alpine

COPY ./scripts/CreateScheme.sql /docker-entrypoint-initdb.d

COPY ./scripts/InsertData.sql /docker-entrypoint-initdb.d

ENV POSTGRES_DB=$POSTGRES_DB \
   POSTGRES_USER=$POSTGRES_USER \
   POSTGRES_PASSWORD=$POSTGRES_PASSWORD