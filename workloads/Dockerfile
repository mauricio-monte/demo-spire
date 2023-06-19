FROM ubuntu:23.10

WORKDIR /workloads

COPY . .

RUN useradd server-workload-user
RUN useradd client-workload-user
RUN useradd unnauthorized-user
RUN useradd spiffe-watcher-user
