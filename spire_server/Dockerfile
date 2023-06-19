FROM ubuntu:23.10

WORKDIR /server

COPY . .

ENTRYPOINT [ "./spire-server", "run", "-config", "./server.conf",  "-logFile", "logs.txt", "&" ]

EXPOSE 10001
