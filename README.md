Konfiguracja i uruchomienie projektu:

1. Wejdź stworzonego katalogu.

2. W katalogu dodaj następujące pliki wraz odpowiednią zawartością:

docker-compose.yml:

```
version: '3.8'

services:
  fastqc:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: fastqc_container
    volumes:
      - fastqc_results:/results
      - ./input:/input:ro
    networks:
      - fastqc_network
    environment:
      JAVA_HOME: /usr/lib/jvm/java-11-openjdk-amd64
      CLASSPATH: /usr/local/FastQC:/usr/local/FastQC/htsjdk.jar:/usr/local/FastQC/jbzip2-0.9.jar:/usr/local/FastQC/cisd-jhdf5.jar
    command: /input/SRR8786200_1.fastq.gz -o /results

  nginx:
    image: nginx:latest
    container_name: nginx_container
    volumes:
      - fastqc_results:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    ports:
      - "8080:80"
    networks:
      - fastqc_network

volumes:
  fastqc_results:

networks:
  fastqc_network:
```

Dockerfile:

```
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    openjdk-11-jre-headless unzip perl wget && \
    wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.12.1.zip && \
    unzip fastqc_v0.12.1.zip && \
    chmod +x FastQC/fastqc && \
    mv FastQC /usr/local/ && \
    ln -s /usr/local/FastQC/fastqc /usr/local/bin/fastqc && \
    apt-get clean && rm -rf /var/lib/apt/lists/* fastqc_v0.12.1.zip

ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ENV PATH=$JAVA_HOME/bin:$PATH
ENV CLASSPATH=/usr/local/FastQC:/usr/local/FastQC/htsjdk.jar:/usr/local/FastQC/jbzip2-0.9.jar:/usr/local/FastQC/cisd-jhdf5.jar

ENTRYPOINT ["fastqc"]
CMD ["--help"]

ENTRYPOINT ["fastqc"]
CMD ["--help"]
```

nginx.conf:

```
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index SRR8786200_1_fastqc.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

3. Utwórz katalog input i umieść tam plik FASTQ SRR8786200_1.fastq.gz
4. Zbuduj i uruchom projekt używając poniższego poelcenia:
docker compose up --build
5. Wyniki:
Po uruchomieniu kontenerów:
- FastQC przeanalizuje plik SRR8786200_1.fastq.gz i zapisze wyniki w wolumenie fastqc_results.

- NGINX udostępni wyniki analizy pod adresem http://localhost:8080.
