# Base image
FROM ubuntu:20.04

# Labels and Credits
LABEL \
    name="reNgine" \
    author="Yogesh Ojha <yogesh.ojha11@gmail.com>" \
    description="reNgine is a automated pipeline of recon process, useful for information gathering during web application penetration testing."

# Environment Variables
ENV DEBIAN_FRONTEND="noninteractive" \
    DATABASE="postgres"

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Install essentials
RUN apt update -y && apt install -y  --no-install-recommends \
    build-essential \
    cmake \
    firefox \
    gcc \
    git \
    libpq-dev \
    libpq-dev \
    libpcap-dev \
    netcat \
    postgresql \
    python3 \
    python3-dev \
    python3-pip \
    python3-netaddr \
    wget \
    x11-utils \
    xvfb \
    python3-cffi \
    python3-brotli \
    libpango-1.0-0 \
    libpangoft2-1.0-0 \
    geoip-bin \
    geoip-database


# Download and install go 1.20
RUN wget https://golang.org/dl/go1.20.linux-amd64.tar.gz
RUN tar -xvf go1.20.linux-amd64.tar.gz
RUN rm go1.20.linux-amd64.tar.gz
RUN mv go /usr/local

# Download and install massdns
RUN wget https://github.com/blechschmidt/massdns/archive/refs/tags/v1.0.0.tar.gz
RUN tar -xvf v1.0.0.tar.gz
RUN rm v1.0.0.tar.gz
RUN make -C massdns-1.0.0/
RUN mv massdns-1.0.0/bin/massdns /usr/bin/massdns
RUN rm -r massdns-1.0.0/

# Download geckodriver
RUN wget https://github.com/mozilla/geckodriver/releases/download/v0.26.0/geckodriver-v0.26.0-linux64.tar.gz
RUN tar -xvf geckodriver-v0.26.0-linux64.tar.gz
RUN rm geckodriver-v0.26.0-linux64.tar.gz
RUN mv geckodriver /usr/bin

# ENV for Go
ENV GOROOT="/usr/local/go"
ENV PATH="${PATH}:${GOROOT}/bin"
ENV PATH="${PATH}:${GOPATH}/bin"

ENV GOPATH=$HOME/go
ENV PATH="${PATH}:${GOROOT}/bin:${GOPATH}/bin"

# Make directory for app
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Download Go packages
RUN go install -v github.com/hakluke/hakrawler@latest

RUN GO111MODULE=on go install -v -v github.com/bp0lr/gauplus@latest

RUN GO111MODULE=on go install -v github.com/jaeles-project/gospider@latest

RUN go install -v github.com/OWASP/Amass/v3/...@latest

RUN go install -v github.com/ffuf/ffuf@latest

RUN go install -v github.com/tomnomnom/assetfinder@latest
RUN GO111MODULE=on go install -v github.com/tomnomnom/gf@latest
RUN GO111MODULE=on go install -v github.com/tomnomnom/unfurl@latest
RUN GO111MODULE=on go install -v github.com/tomnomnom/waybackurls@latest

RUN GO111MODULE=on go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
RUN GO111MODULE=on go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
RUN GO111MODULE=on go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
RUN GO111MODULE=on go install -v github.com/projectdiscovery/naabu/v2/cmd/naabu@latest
RUN GO111MODULE=on go install -v github.com/projectdiscovery/shuffledns/cmd/shuffledns@latest


# Update Nuclei and Nuclei-Templates
RUN nuclei -update
RUN nuclei -update-templates

# Copy requirements
COPY ./requirements.txt /tmp/requirements.txt
RUN pip3 install --upgrade setuptools pip && \
    pip3 install -r /tmp/requirements.txt

# install eyewitness

RUN python3 -m pip install fuzzywuzzy \
    selenium \
    python-Levenshtein \
    pyvirtualdisplay \
    netaddr

# install dnsvalidator
RUN git clone https://github.com/vortexau/dnsvalidator.git
WORKDIR dnsvalidator
RUN python3 setup.py install
WORKDIR /usr/src/app
RUN rm -r dnsvalidator

# install Cron
RUN apt-get update
RUN apt-get -y install cron

# Add the cron job
RUN crontab -l | { cat; echo "0 0 * * * dnsvalidator -tL https://public-dns.info/nameservers.txt -threads 40 -o /usr/src/app/resolvers.txt"; } | crontab -
RUN cron

# Copy source code
COPY . /usr/src/app/

# httpx seems to have issue, use alias instead!!!
RUN echo 'alias httpx="/go/bin/httpx"' >> ~/.bashrc
