<img height=auto width=100% src="https://raw.githubusercontent.com/RuiSiang/PoW-Shield/main/screenshot.jpg" alt="PoW Shield">
<div align="center">
  <img src="https://github.com/RuiSiang/PoW-Shield/actions/workflows/nodejs-ci.yml/badge.svg">
  <img src="https://github.com/RuiSiang/PoW-Shield/actions/workflows/njsscan-analysis.yml/badge.svg">
  <img src="https://github.com/RuiSiang/PoW-Shield/actions/workflows/codeql-analysis.yml/badge.svg">
  <a href="https://hub.docker.com/r/ruisiang/pow-shield">
    <img src="https://img.shields.io/docker/image-size/ruisiang/pow-shield/latest?label=docker%20image%20size">
  </a>
</div>
<div align="center">
  <img src="https://img.shields.io/github/repo-size/ruisiang/pow-shield?color=orange">
  <a href="https://deepsource.io/gh/RuiSiang/PoW-Shield/?ref=repository-badge">
    <img src="https://deepsource.io/gh/RuiSiang/PoW-Shield.svg/?label=active+issues&show_trend=true&token=yoFuBlRVaXTzIVkgAB6aSUf3">
  </a>
  <a href="https://deepsource.io/gh/RuiSiang/PoW-Shield/?ref=repository-badge">
    <img src="https://deepsource.io/gh/RuiSiang/PoW-Shield.svg/?label=resolved+issues&show_trend=true&token=yoFuBlRVaXTzIVkgAB6aSUf3">
  </a>
</div>

## Description

PoW Shield provides DDoS protection on OSI application layer by acting as a proxy that utilizes proof of work between the backend service and the end user. This project aims to provide an alternative to general captcha methods such as Google's ReCaptcha that has always been a pain to solve. Accessing a web service protected by PoW Shield has never been easier, simply go to the url, and your browser will do the rest of the verification automatically for you.

PoW Shield aims to provide the following services bundled in a single webapp / docker image:

- proof of work authentication
- ratelimiting and ip blacklisting
- web application firewall

## Stargazers

[![Stargazers repo roster for PoW Shield](https://reporoster.com/stars/ruisiang/pow-shield)](https://github.com/ruisiang/pow-shield/stargazers)

[Story on Medium](https://ruisiang.medium.com/pow-shield-application-layer-proof-of-work-ddos-filter-4fed32465509 'PoW Shield: Application Layer Proof of Work DDoS Filter')

[Featured on Pentester Academy TV](https://youtu.be/zeNKUDR7_Jc 'The Tool Box | PoW Shield')

## Features

- Web Service Structure
- Proxy Functionality
- PoW Implementation
- Dockerization
- IP Blacklisting
- Ratelimiting
- Unit Testing
- WAF Implementation
- Multi-Instance Syncing (Redis)

## How it Works

So basically, PoW Shield works as a proxy in front of the actual web app/service. It conducts verification via proof-of-work and only proxies authorized traffic through to the actual server. The proxy is easily installable, and is capable of protecting low security applications with a WAF.

Here’s what happens behind the scenes when a user browses a PoW Shield-protected webservice:

1. The server generates a random hex-encoded “prefix” and sends it along with the PoW Shield page to the client.
2. Browser JavaScript on the client side then attempts to brute-force a “nonce” that when appended with the prefix, can produce a SHA256 hash with the number of leading zero-bits more than the “difficulty” D specified by the server. i.e. SHA256(prefix + nonce)=0…0xxxx (binary, with more than D leading 0s)
3. Client-side JavaScript then sends the calculated nonce to the server for verification, if verification passes, the server generates a cookie for the client to pass authentication.
4. The server starts proxying the now authenticated client traffic to the server with WAF filtering enabled.

## Configuration

You can configure PoW Shield via the following methods.

- nodejs: .env (example: .env.example)
- docker-compose: docker-compose.yaml (example: docker-compose.example.yaml)
- docker run: -e parameter

### Environmental Variables

| Variable                     | Type       | Default         | Description                                                                                                                                                       |
| ---------------------------- | ---------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PORT                         | General    | 3000            | port that PoW Shield listens to                                                                                                                                   |
| SESSION_KEY                  | General    |                 | secret key for cookie signatures, use a unique one for security reasons, or anyone can forge your signed cookies                                                  |
| BACKEND_URL                  | General    |                 | location to proxy authenticated traffic to, IP and URLs are both accepted(accepts protocol://url(:port) or protocol://ip(:port))                                  |
| DATABASE_HOST                | Redis      | 127.0.0.1       | redis service host                                                                                                                                                |
| DATABASE_PORT                | Redis      | 6379            | redis service port                                                                                                                                                |
| DATABASE_PASSWORD            | Redis      | null            | redis service password                                                                                                                                            |
|                              |
| POW                          | PoW        | on              | toggles PoW functionality on/off (if not temporary switched off, why use this project at all?)                                                                    |
| NONCE_VALIDITY               | PoW        | 60000           | specifies the maximum seconds a nonce has to be submitted to the server after generation(used to enforce difficulty change and filter out stale nonces)           |
| INITIAL_DIFFICULTY           | PoW        | 13              | initial difficulty, number of leading 0-bits in produced hash (0:extremely easy ~ 256:impossible, 13(default) takes about 5 seconds for the browser to calculate) |
| RATE_LIMIT                   | Rate Limit | on              | toggles ratelimit functionality on/off                                                                                                                            |
| RATE_LIMIT_SAMPLE_MINUTES    | Rate Limit | 60              | specifies how many minutes until statistics reset for session/ip                                                                                                  |
| RATE_LIMIT_SESSION_THRESHOLD | Rate Limit | 100             | number of requests that a single session can make until triggering token revocation                                                                               |
| RATE_LIMIT_BAN_IP            | Rate Limit | on              | toggles ip banning functionality on/off                                                                                                                           |
| RATE_LIMIT_IP_THRESHOLD      | Rate Limit | 500             | number of requests that a single session can make until triggering IP ban                                                                                         |
| RATE_LIMIT_BAN_MINUTES       | Rate Limit | 15              | number of minutes that IP ban persists                                                                                                                            |
| WAF                          | WAF        | on              | toggles waf functionality on/off                                                                                                                                  |
| WAF_URL_EXCLUDE_RULES        | WAF        |                 | exclude rules to check when scanning request url, use ',' to seperate rule numbers, use '-' to specify a range (eg: 1,2-4,5,7-10)                                 |
| WAF_HEADER_EXCLUDE_RULES     | WAF        | 14,33,80,96,100 | exclude rules to check when scanning request header, use ',' to seperate rule numbers, use '-' to specify a range (eg: 1,2-4,5,7-10)                              |
| WAF_BODY_EXCLUDE_RULES       | WAF        |                 | exclude rules to check when scanning request body, use ',' to seperate rule numbers, use '-' to specify a range (eg: 1,2-4,5,7-10)                                |

## Usage

### Nodejs

#### Prerequisites

- Docker ^19.0.0
- Nodejs ^14.0.0

```bash
# Clone repository
git clone https://github.com/RuiSiang/PoW-Shield.git

# Install dependencies
npm install

# Configure settings
cp -n .env.example .env
# Edit .env
nano .env

# Transpile
npm run build

#############################################
# Run with db (redis), recommended & faster #
# install redis first                       #
# sudo apt-get install redis-server         #
#############################################
npm start
#############################################

#############################################
#        Run without db (mock redis)        #
#############################################
npm run start:standalone # linux
npm run start:standalone-win # windows
#############################################

# Test functionalities(optional)
npm test

```

### Docker ([repo](https://hub.docker.com/repository/docker/ruisiang/pow-shield))

```bash
####################################################
# Docker run with db (redis), recommended & faster #
####################################################
docker run -p 3000:3000 -e BACKEND_URL="http://example.com" -d ruisiang/pow-shield
####################################################

####################################################
#        Docker run without db (mock redis)        #
####################################################
docker run -p 3000:3000 -e BACKEND_URL="http://example.com" -e NODE_ENV="standalone" -d ruisiang/pow-shield
####################################################

####################################################
#                  Docker Compose                  #
####################################################
# Copy docker-compose.example.yaml
cp -n docker-compose.example.yaml docker-compose.yaml
# Edit docker-compose.yaml
nano docker-compose.yaml

# Start the container
docker-compose -f docker-compose.yaml up
####################################################
```

## References

- Proof-of-work by Fedor Indutny (PoW utility functions)
- Shadowd by Zesecure (WAF rules)

## License

MIT
