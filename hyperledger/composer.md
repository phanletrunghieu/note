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
