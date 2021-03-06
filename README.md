#### Latest Deployment:

- AMAEnsClient: 0x3E8De8e39126584F61B5Ac65674483e20BaEFCc0
- PublicResolver: 0x88eD73819541DC273Db1F347354280d52A73ffFC
- ENSRegistry: 0x2f75d36C37fB32456AfC0c6bb8508A4aDB49f067
- BaseRegitrar: 0xb6e3c97c77BbeC03DEb77c4755d4B2aaBb2cC79b

Note:
TO find the owner of the subdomain, use 

```
getNodeHash(string memory _label) 
```
function on AMAENSClient to get the nodeHash of the label, for example if you want to get nodehash of
amafansofficial.amafans, then use anafansofficial as _label to get the nodeHash which is 
0x35d73f356b889ed2a58491b31f9e5babb6af79aed24a6dd80cfc4cccb25a34ab
use this value of ENSregitsry contract to get the owner of the domain.







This is the fork of the official ENS Repository. https://github.com/ensdomains .
Because of the unavailability of the ENS repository on AVAX, We created this fork to let our users have access
to the subdomains, To have an identity to be used on AMA.Fans platform.


#### NameHash of amafans: 0x2c36ca5c2f315c648f49b490565ed094e37a6e8d230039597a7827db6fbea638: 



- #### ENSRegistry takes care of the root domain, its subdomains (in this case only one amafans) 
    and all the other subdomains of subdomains and so on. WHoever deploys this contract becomes the 
    owner of rootDomain 0X0 ``` records[0x0].owner = msg.sender; ```

- #### BaseRegistrar which actually owns the root domain and has the relevant permissions to control the rootDomain.It has the 
    relevant permissions to register, renew, reclaim  and controllers.

- #### _RootDomain_: 0x0000000000000000000000000000000000000000000000000000000000000000
    - #### _SubDomain_ :  0x2c36ca5c2f315c648f49b490565ed094e37a6e8d230039597a7827db6fbea638 (Namehash of amafans)
        - #### _AllSubDomains: 0x22eefbbc1c0b5e5abcfa458ff05bb36637914d1b055acf7c62a6a93c2210e8c6 (labelHash of amafans)
            - #### _RestOfTheSubmainsUnderit_ : calculate the labelhash of the string and call setSubnodeOwner.


- #### 		await ens.setSubnodeOwner('0x0', sha3('amafans'), registrar.address);
        this ensures that amafans is actually a subdomain of the rootDomain which is 0X0. and every other 
        domain will be created as a subdomain of amafans.


#### Steps to deploy ENS on Avalance Testnet:

1. Deploy ENSRegistry.sol (use the OwnerAccount)
2. Deploy BaseregistrarImplementation.sol with base node (Namehash of amafans) i.e 0x2c36ca5c2f315c648f49b490565ed094e37a6e8d230039597a7827db6fbea638
        ```
		    registrar = await BaseRegistrar.new(ens.address, namehash.hash('amafans'), {from: ownerAccount});
		
        ```
    Use the OwnerAccount

3. Add a subnode owner on the ENS contract, this will make the registrar contract the owner of the .amafans domain on the ens contract.
   Use labelhash of the amafans for this. this is saying that the root node is 0x0 and the subnode is sha3(amafans). This has to be 
   called from the address who deployed ens registry contract.  this also sets the owner of the amafans as the Baseregistrar. 
   All actions for creating subdomains on .amafans has to called by baseregistrar function.

    ```
		await ens.setSubnodeOwner('0x0', sha3('amafans'), registrar.address);
    ```
    sha3('amafans') is the labelhash: 0x22eefbbc1c0b5e5abcfa458ff05bb36637914d1b055acf7c62a6a93c2210e8c6

3. Testing the deployed contracts
    - #### First you need to add a controller to Baseregister so that address can create a new subdomain on amafans.
            So, the address who deployed the Baseregistrar must call addController with another address2
            lets call it address_2(will create subdomain):

            ```
                await registrar.addController(controllerAccount, {from: ownerAccount});
            ```
            after this address_2 can call register function on the BaseRegistrar and a subdomain on ENS will
            be created. 
    - #### lets create a new subdomain test.amafans.
            ```
                  
            function getNodeHash(string memory _label) external pure returns (bytes32,bytes32,uint256){
                bytes32 label = keccak256(bytes(_label));
                uint256 tokenId = uint256(label);
                return (label, keccak256(abi.encodePacked(BASENODE, label)), tokenId);

            }
            BASENODE is namehash of AMAFans.
            ```
            tokenId(test) = 70622639689279718371527342103894932928233838121221666359043189029713682937432
	    nodehash(test) = 0xf90a0f90ddc34eaa5169bdd6a19e95ad0aaae90dfe3fe74b216907293f394ecf
	    owner = "0x0000000000000000000000000000000000000001"
            duration = 31536000
            Call register from address_2
    - #### lets check on ENSRegistry if this subdomain has been created or not.
            use getNodeHash function with input as "testtest" and use this ```keccak256(abi.encodePacked(BASENODE, label))```
            to ge the nodehash. With this nodeHash call recordExists and you will seee the owner as same as above.


    
5. Deploy PublicResolver with ENSRegistry contract address and WRAPPERADDRESS = 0x0000000000000000000000000000000000000000.
7. Call setResolver on the BaseregistrarImplementation with the owner of amafans node.
8. Deploy AMAENSCLient.sol with the BaseregistrarImplementation address, PublicResolver address and duration of your liking.

9. Add a controller (addController) on the BaseregistrarImplementation contract with AMAENSCLient contract address as an input, which means that 
AMAENSCLient contract can take actions on behalf of the BaseregistrarImplementation contract. This is required because before this 
only baseregistrar could create subdomains for amafans on ENSregistry contract.

11. setController function has to be called on the AMAENSCLient contract with operator as the address of AMACLClient 
12. The call registerNode function on the AMAENSCLient contract with the name and the owner (Only owner of the contract can call the same).

13. The resolver will the default publicResolver.



- ### Steps for  deployment of contracts
- ##### check if you are going to deploy on mainnet and testnet. Based on the network, You may
        want to change the RPC_URL, OWNER_PRIVATE_KEY, FEECCOLLECTOR_PRIVATE_KEY, OPERATOR_PRIVATE_KEY
- #### run after_cl_client_deploy script only when the AMAClClient deployment is already done, This script 
        just add AMACLClient address as an operator on the AMAENSClient so that AMACLClient can take actions 
        on the root domain.
- #### You dont need to have AMACLClient address while running deploy.js
- #### Make sure the OWNER_PRIVATE_KEY, FEECCOLLECTOR_PRIVATE_KEY, OPERATOR_PRIVATE_KEY are same accross 
        all the repositories.
