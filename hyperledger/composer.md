## Install

- node version 8.16.0 ()
- npm install -g composer-cli@0.20.8 composer-rest-server@0.20.8 generator-hyperledger-composer@0.20.8 composer-playground@0.20.8 yo

## Setup network

- mkdir fabric-dev-servers && cd fabric-dev-servers
- curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
- tar -xvf fabric-dev-servers.tar.gz

- cd fabric-dev-servers
- FABRIC_VERSION=hlfv12 ./downloadFabric.sh
- FABRIC_VERSION=hlfv12 ./startFabric.sh

## Import card

Will import at ~/.composer/

- FABRIC_VERSION=hlfv12 ./createPeerAdminCard.sh

Stop with:
- FABRIC_VERSION=hlfv12 ./stopFabric.sh

## Generate compose project

yo hyperledger-composer:businessnetwork

## Generate a business network archive (.bna file)

composer archive create -t dir -n .

## Deploy to network

composer network install --card PeerAdmin@hlfv1 --archiveFile tutorial-network@0.0.1.bna

composer network start --networkName tutorial-network --networkVersion 0.0.1 --networkAdmin admin --networkAdminEnrollSecret adminpw --card PeerAdmin@hlfv1 --file networkadmin.card

composer card import --file networkadmin.card

## Show composer card

composer card list

## Generate REST Server

composer-rest-server

## Enable OAuth2

Dockerfile
```Dockerfile
FROM hyperledger/composer-rest-server:0.20.8
RUN npm install --production loopback-connector-mongodb passport-google-oauth2 && \
npm cache clean --force && \
ln -s node_modules .node_modules
```

docker-compose.yml
```yml
version: '3'

services:
  app:
    build: .
    restart: always
    ports:
      - "3000:3000"
    volumes:
      - ~/.composer:/home/composer/.composer
    environment:
      COMPOSER_CARD: admin@tutorial-network
      COMPOSER_NAMESPACES: never
      COMPOSER_AUTHENTICATION: 'true'
      COMPOSER_MULTIUSER: 'true'
      COMPOSER_PROVIDERS: '{
        "google": {
          "provider": "google",
          "module": "passport-google-oauth2",
          "clientID": "312039026929-t6i81ijh35ti35jdinhcodl80e87htni.apps.googleusercontent.com",
          "clientSecret": "Q4i_CqpqChCzbE-u3Wsd_tF0",
          "authPath": "/auth/google",
          "callbackURL": "/auth/google/callback",
          "scope": "https://www.googleapis.com/auth/plus.login",
          "successRedirect": "/",
          "failureRedirect": "/"
        }
      }'
      COMPOSER_DATASOURCES: '{
        "db": {
          "name": "db",
          "connector": "mongodb",
          "host": "mongo"
        }
      }'

  mongo:
    image: mongo:4.2.0-rc1
    restart: always
    ports:
      - "27017:27017"

networks:
  default:
    external:
      name: composer_default
```

Edit host use in docker

sed -e 's/localhost:7051/peer0.org1.example.com:7051/' -e 's/localhost:7053/peer0.org1.example.com:7053/' -e 's/localhost:7054/ca.org1.example.com:7054/'  -e 's/localhost:7050/orderer.example.com:7050/'  < $HOME/.composer/cards/restadmin@tutorial-network/connection.json  > /tmp/connection.json && cp -p /tmp/connection.json $HOME/.composer/cards/restadmin@@tutorial-network/

## Create new participant

- composer participant add -c admin@tutorial-network -d '{"$class":"org.example.mynetwork.SampleParticipant","participantId":"part1", "firstName":"Jo","lastName":"Doe"}'
- composer identity issue -c admin@tutorial-network -f jdoe.card -u jdoe -a "resource:org.example.mynetwork.SampleParticipant#part1"
- composer card import -f jdoe.card

Edit cart > Export > Import wallet

sed -e 's/localhost:7051/peer0.org1.example.com:7051/' -e 's/localhost:7053/peer0.org1.example.com:7053/' -e 's/localhost:7054/ca.org1.example.com:7054/'  -e 's/localhost:7050/orderer.example.com:7050/'  < $HOME/.composer/cards/restadmin@tutorial-network/connection.json  > /tmp/connection.json && cp -p /tmp/connection.json $HOME/.composer/cards/restadmin@@tutorial-network/

composer card export -f jdoe_exp.card -c jdoe@tutorial-network
