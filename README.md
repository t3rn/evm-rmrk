# RMRK Solidity

A set of Solidity contracts for RMRK.app.

## Core

Multi resource and nesting ability.

Use Core2.sol, Core.sol was an attempt at making it ERC721 BC but I ran into difficulties and decided to build from the ground up instead and add BC later.

## Settings

> TBD

A storage contract containing values like the RMRK Fungibilization deposit (how many tokens you need to make an NFT into a collection of fungibles) and other governance-settable values.

## Equip

> TBD

Equipping and Base entity.

## Emotable

> TBD

Emotes are useful, but very expensive to store. Some important considerations are documented here: https://github.com/rmrk-team/pallet-emotes and here: https://hackmd.io/JjqT6THTSoqMj-_ucPEJAw?view - needs storage oprimizations considerations vs wasting gas on looping. Benchmarking would be GREAT.

## Fractional

> TBD

Turning NFTs into fractional tokens after a deposit of RMRK.
The deposit size should be read from Settings.

## Logic

> TBD

JSONlogic for front-end variability based on on-chain values.
Logic should go into a Logic field of the NFT's body, and is executed exclusively in the client.

## Harberger

> TBD

An extension for the contracts to make them Harberger-taxable by default, integrating the selling and taxing functionality right into the NFT's mint. This does mean the NFT can never not be Harb-taxed, but there can be an on-off flag for this that the _ultimate owner_ (a new owner type?) can flip.

---

## Develop

Just run `npx hardhar compile` to check if it works. Refer to the rest below.

This project demonstrates an advanced Hardhat use case, integrating other tools commonly used alongside Hardhat in the ecosystem.

The project comes with a sample contract, a test for that contract, a sample script that deploys that contract, and an example of a task implementation, which simply lists the available accounts. It also comes with a variety of other tools, preconfigured to work with the project code.

Try running some of the following tasks:

```shell
npx hardhat accounts
npx hardhat compile
npx hardhat clean
npx hardhat test
npx hardhat node
npx hardhat help
REPORT_GAS=true npx hardhat test
npx hardhat coverage
npx hardhat run scripts/deploy.ts
TS_NODE_FILES=true npx ts-node scripts/deploy.ts
npx eslint '**/*.{js,ts}'
npx eslint '**/*.{js,ts}' --fix
npx prettier '**/*.{json,sol,md}' --check
npx prettier '**/*.{json,sol,md}' --write
npx solhint 'contracts/**/*.sol'
npx solhint 'contracts/**/*.sol' --fix
```

## Etherscan verification

To try out Etherscan verification, you first need to deploy a contract to an Ethereum network that's supported by Etherscan, such as Ropsten.

In this project, copy the .env.example file to a file named .env, and then edit it to fill in the details. Enter your Etherscan API key, your Ropsten node URL (eg from Alchemy), and the private key of the account which will send the deployment transaction. With a valid .env file in place, first deploy your contract:

```shell
hardhat run --network ropsten scripts/sample-script.ts
```

Then, copy the deployment address and paste it in to replace `DEPLOYED_CONTRACT_ADDRESS` in this command:

```shell
npx hardhat verify --network ropsten DEPLOYED_CONTRACT_ADDRESS "Hello, Hardhat!"
```

# Performance optimizations

For faster runs of your tests and scripts, consider skipping ts-node's type checking by setting the environment variable `TS_NODE_TRANSPILE_ONLY` to `1` in hardhat's environment. For more details see [the documentation](https://hardhat.org/guides/typescript.html#performance-optimizations).

## `solang` + `substrate` adaption 

Branch `rmrk-wasm` features a `substrate` wasm build version of the `RMRKResource` contract. Required a little adaption as `assembly` and other calls are unsupported with solang making the `Address` library unusable. Thus `RMRKNestable#_mintNest` does not longer include a `require(to.isContract(), "Is not contract");` condition. Other modifications required to make the code compile were casts from `bytes8` to `bytes32` in `RMRKResource`, a slight adaption of the `try` block in `RMRKNestable#findRootOwner`, and declaring size variables in the `Strings` library as `uint32`s rather than `uint256`s.

**installing `llvm13` + `solang@v0.1.10`**

``` bash
curl -fL \
  -o $HOME/llvm13.0.tar.xz  \
  https://github.com/hyperledger-labs/solang/releases/download/v0.1.10/llvm13.0-linux-x86-64.tar.xz
cd $HOME
tar Jxvf llvm13.0.tar.xz
rm llvm13.0.tar.xz
touch $HOME/.bashrc
echo 'export "PATH=$HOME/llvm13.0/bin:$PATH"' >> $HOME/.bashrc
echo 'export LLVM_SYS_130_PREFIX=$HOME/llvm13.0' >> $HOME/.bashrc
curl -fL \
  -o $HOME/solang \
  https://github.com/hyperledger-labs/solang/releases/download/v0.1.10/solang-linux-x86-64
chmod +x $HOME/solang
mv $HOME/solang /usr/local/bin/solang
```

**compiling**

``` bash
solang --target substrate -o ./artifacts/ ./contracts/RMRK/RMRKResource.sol
```