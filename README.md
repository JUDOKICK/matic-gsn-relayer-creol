# Matic GSNv1 Relayer - by Creol.io
The following is comprehensive instructions for setting up a GSN Relayer on Matic, but this can be extended to ANY Ethereum based network as the process is exactly the same.

## Pre-requisites

1. Domain purchased that you own. AWS, GoDaddy, Namecheap,etc is all acceptable. You will need to know how to modify your DNS records.

2. Cloud Docker service or system where you can expose a docker container.

3. Funds in your Matic / Eth wallet to pay for registration of your Relayer on the network.

## Installation

1. First register a subdomain on your DNS records like "gsn-relayer.YOURDOMAIN.COM" and point it to a generic IP like 127.0.0.1

2. Next Fork this Repo and modify the docker-compose.yml to match your subdomain, and add your RPC node URL.

    You can use a private Infura node or a publicly available node like: https://rpc-mumbai.matic.today
    ```javascript
           services:
             https-portal:
               image: steveltn/https-portal:1
               ports:
                 - '80:80'
                 - '443:443'
               restart: always
               environment:
                 DOMAINS: 'gsn-relayer.YOURDOMAIN.COM -> http://gsn1'
                 STAGE: 'production'
           
             gsn1:
               image: dmihal/gsn-relay-xdai
               volumes:
                 - /etc/gsn1-data:/app/data
               environment:
                 URL: https://gsn-relayer.YOURDOMAIN.com
                 LOCAL_PORT: 80
                 NODE_URL: https://RPC_NODE_URL_HERE
                 RELAY_HUB: "0xD216153c06E857cD7f72665E0aF1d7D82172F494"
                 GAS_PRICE_PERCENT: 1
    ```

3. Next deploy your forked docker-compose on a docker cloud instance like Jelastic/AWS ECS etc. 

4. Take the deployed IP address of the instance and update your DNS record from step 1.
    * It'll resemble this
    ```CNAME || gsn-relayer.YOURDOMAIN.com  || YOUR_DOCKER_IP```
 5. To check if your relayer is running correctly and the DNS is registered, open
    https://gsn-relayer.YOURDOMAIN.com/getaddr 
    
 6. Now you will have to register your Relayer on the smart contract. All GSNv1 contracts have the same deployment address regardless of network
    ```javascript
        0xD216153c06E857cD7f72665E0aF1d7D82172F494
    ```
 7. You will also have to transfer some funds to the Relayer's address so it can pay some txns. Min is 1 ETH i believe.
   
 8. Using remix.ethereum.org or your own implementation, compile the RelayHub.sol contract provided here and use the ```stake``` function
    * You will need to get the Relayer address from the output of the docker container (you can use ```docker logs GSNCONTAINERID``` to get it)
    * Minimum you must stake is 1 ETH / or 1 MATIC and the default delay set is ``604800`` (1 week approx)
    
 9. Your Relayer at this point should detect that it is in fact registered and begin accepting relays
 
## Running GSN Transactions

To be able to run GSN capable contracts, your smart contract must have inherited the ```GSNRecipient.sol``` contract.

Then you will have to deposit funds to the ```RelayHub``` contract for your deployed contracts address.

After this, any function that gets called using GSN relayers will deduct from this balance. 


