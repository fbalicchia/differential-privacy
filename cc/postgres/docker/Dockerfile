FROM postgres:11 AS build

ENV PROTOC_VERSION=1.3

ENV DF_COMMIT_ID=20f80aaf40b6d71c73c3b21e2ed77cbe54dbc1fc

# Install the packages which will be required to get everything to compile
RUN apt-get update \
    && apt-get install -f -y --no-install-recommends \
        software-properties-common \
        build-essential \
        pkg-config \
        git \
        curl \
        libreadline-dev \ 
        bison \
        flex \
        postgresql-server-dev-$PG_MAJOR \
    && add-apt-repository "deb http://ftp.debian.org/debian testing main contrib" \
    && apt-get update && apt-get install -f -y --no-install-recommends \
        libprotobuf-c-dev=$PROTOC_VERSION.* \
    && rm -rf /var/lib/apt/lists/*

#Install Bazel to build code
RUN echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list \
  && curl https://bazel.build/bazel-release.pub.gpg | apt-key add -


RUN apt-get update \
  && apt-get install -y bazel \
  && rm -rf /var/lib/apt/lists/*


##Download differential privacy module


RUN git clone https://github.com/google/differential-privacy.git -b master --single-branch /tmp/differential-privacy \
    && cd /tmp/differential-privacy \
    && git checkout $DF_COMMIT_ID \
    && cd /tmp/differential-privacy/cc \
    && bazel build postgres/anon_func.so 


#COPY ./differential-privacy /tmp/differential-privacy
#RUN cd /tmp/differential-privacy/cc;bazel build postgres/anon_func.so


FROM spilo:local


COPY --from=build /tmp/differential-privacy/cc/bazel-bin/postgres/anon_func.so  /usr/lib/postgresql/11/lib/
COPY --from=build /tmp/differential-privacy/cc/postgres/anon_func.control /usr/share/postgresql/11/extension/
COPY --from=build /tmp/differential-privacy/cc/postgres/anon_func--1.0.0.sql /usr/share/postgresql/11/extension/

#Copy Dataset sample 
COPY --from=build /tmp/differential-privacy/cc/postgres/fruiteaten.csv  /home/postgres/
COPY --from=build /tmp/differential-privacy/cc/postgres/shirts.csv  /home/postgres/
