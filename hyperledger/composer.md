## Install

- node version 8.16.0 ()
- npm install -g composer-cli@0.20.8 composer-rest-server@0.20.8 generator-hyperledger-composer@0.20.8 composer-playground@0.20.8 yo

## Setup network

- mkdir fabric-dev-servers && cd fabric-dev-servers
- curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
- tar -xvf fabric-dev-servers.tar.gz

- cd fabric-dev-servers
- FABRIC_VERSION=hlfv11 ./downloadFabric.sh
- FABRIC_VERSION=hlfv11 ./startFabric.sh
- FABRIC_VERSION=hlfv11 ./createPeerAdminCard.sh

Stop with:
- FABRIC_VERSION=hlfv11 ./createPeerAdminCard.sh

## Generate compose project

yo hyperledger-composer:businessnetwork

## Generate a business network archive (.bna file)

composer archive create -t dir -n .

## Deploy to network

composer network install --card PeerAdmin@hlfv1 --archiveFile tutorial-network@0.0.1.bna
