# Set up
Download and install Docker and Hyperledger Fabric along with the nodeJS SDK kit. 1) Define the peers, orderers and organisations in the network by generating the crypto certificates for each of them. This can be done by editing the crypto-config.yml file. After this, run the following command in the directory where the cryptogen bash is present<br />
&nbsp;&nbsp;&nbsp;&nbsp;./cryptogen generate –config=./crypto-config.yaml<br />
In the configtx.yml file, we have to define the orderer organisations and the peer organisations path to the certificates (MSP- membership service provider certificate path generated in the previous step). We will also have to define the anchor peers. The block size of each transaction in the network for the orderer is defined and limited in this config file. Each organisation will have one anchor peer to facilitate the communication between organisations. The type of the orderer (ordering service) is also defined. Now, we have to generate the orderer genesis block by the below command
&nbsp;&nbsp;&nbsp;&nbsp;./configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block<br />
Ordering service in the network will fail if the genesis block is not present. Once the genesis block is created, we have to create a channel so that peers can communicate with eachother. So run the following command<br />
&nbsp;&nbsp;&nbsp;&nbsp;./configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID {your channel 
&nbsp;&nbsp;&nbsp;&nbsp;name}<br />
To generate the anchor peers for organisation Org1MSP/ to define the anchor peer for that organisation, execute this command along with the channel name that's already created<br />
&nbsp;&nbsp;&nbsp;&nbsp;./configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID 
&nbsp;&nbsp;&nbsp;&nbsp;{your channel name} -asOrg Org1MSP<br />
Edit the docker-compose-cli and the docker-compose-base yaml files to start the approriate number of peers and the organisations. Check that different ports are exposed for each service. After the configurations are set, run this -
&nbsp;&nbsp;&nbsp;&nbsp;CHANNEL_NAME={your channel name} TIMEOUT=10 IMAGE_TAG=latest docker-compose -f docker-compose-cli.yaml &nbsp;&nbsp;&nbsp;&nbsp;up -d<br />
Run docker ps to see that the peers as well as the cli container has started.<br />
Now we need to instantiate the channel(generate the channel block) by using the channel configuration transaction(channel.tx) that we created earlier<br />
&nbsp;&nbsp;&nbsp;&nbsp;peer channel create -o orderer.hyperledger.testing.com:7050 -c <your channel name> -f ./channel-artifacts/channel.tx --tls --cafile &nbsp;&nbsp;&nbsp;&nbsp;/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/hyperledger.testing.com/orderers/orde
&nbsp;&nbsp;&nbsp;&nbsp;rer.hyperledger.testing.com/msp/tlscacerts/tlsca.hyperledger.testing.com-cert.pem<br/>
&nbsp;&nbsp;&nbsp;&nbsp;peer channel join -b {your channel name}.block<br />
This will make the ordering service and the peer join the channel.<br/>
Next, we have to make other peers join the channel. We can do this by overriding the CLI properties defined through this command<br/>
&nbsp;&nbsp;&nbsp;&nbsp;CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.hyperledger.testing.com/users/Admin@org2.hyperledger.testing.com/msp
CORE_PEER_LOCALMSPID=Org2MSP &nbsp;&nbsp;&nbsp;&nbsp;CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.hyperledger.testing.com/peers/peer0.org2.hyperledger.testing.com/tls/ca.crt peer channel join -b {your channel name}.block<br />
We'll have to update the channel definition to define the anchor peer for the organization i.e let the channel know of the anchor peers.<br />
&nbsp;&nbsp;&nbsp;&nbsp;peer channel update -o orderer.hyperledger.testing.com:7050 -c myanchorchannel -f ./channel &nbsp;&nbsp;&nbsp;&nbsp;artifacts/Org1MSPanchors.tx --tls –cafile &nbsp;&nbsp;&nbsp;&nbsp;/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/hyperledger.testing.com/orderers/orde
&nbsp;&nbsp;&nbsp;&nbsp;rer.hyperledger.testing.com/msp/tlscacerts/tlsca.hyperledger.testing.com-cert.pem<br />
Do this for each organisation in the network.<br/>
