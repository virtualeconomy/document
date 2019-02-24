---
SuperNOdes Deployment
---
### Introduction
In VEE blockchain system, only supernodes have the right to minting blocks and getting VVV rewards. In this wiki we will give you details on deploying a supernode. Additionally, a few recommendations are provided to increase security of the blockchain and the safety of your coins. We assume you have read [[How to install VSYS testnet Node]] and [[VSYS Testnet Node Configuration File]]. You are recommended to deploy a supernode on testnet first.
### Prerequisites
* A VEE wallet (`wallet.dat` file： [wallet generator](https://github.com/virtualeconomy/vsys-wallet-generator))
* 50k VSYS + 1 million _`effective`_ balance (regular balance + leased in - leased out)
### Contend a minting slot
Once you meet the prerequisites, you are able to contend for a minting slot. Check the slot information from our public [API](http://3.0.221.137:9922) or your local node at _/consensus/allSlotInfo_. Choose a minting slot that you want to contend. Then you could contend the minting slot you chosen through _/spos/contend_ or _/spos/broadcast/contend_. The contended slot can be empty or occupied. When you contend, the system will charge you 50k VSYS as contending fee and check your remaining effective balance which should no less than 1 million for a successful contention. If the contended slot was empty, your account would occupy that slot successfully. If the contended slot was originally occupied by another account, the system would compare the minting average balance (MAB) of your account with that account, and your contention would  be successful only when your MAB is higher. If you failed due to insufficient MAB, your contending fee would **NOT** be returned. 
* Note: Now we have opened 15 slots in mainnet, only those slots with the ID numbers that can exactly divided by 4 are available.
### Wallet and reward settings
To make your VSYS secure, we recommend you set your account to be a `cold minting` status.
* Store your huge amount of VSYS in your cold wallet and lease money to the account that you are going to contend a minting slot
* Set the reward address to be your cold wallet address (refer to `Minter settings` at [V Systems Testnet Node Configuration File](https://vsys.readthedocs.io/en/latest/testconf.html), [V Systems Mainnet Node Configuration File](https://vsys.readthedocs.io/en/latest/mainconf.html)
### API settings
We strongly recommend you setup a standby node and disable API of your minter (refer to REST API settings at [V Systems Testnet Node Configuration File](https://vsys.readthedocs.io/en/latest/testconf.html), [V Systems Mainnet Node Configuration File](https://vsys.readthedocs.io/en/latest/mainconf.html)
