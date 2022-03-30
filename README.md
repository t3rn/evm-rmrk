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

Just run `npx hardhat compile` to check if it works. Refer to the rest below.

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

***

## `solang` + `substrate` adaption 

Branch `dev-t3rn-2` features a `solang` compatible `substrate` wasm targeting version of the `RMRK` contracts.

Currently based upon `dev@ab1aaf9`.

To check solang specific modifications, please compare branches.

### Try it out

#### Installing `llvm13.0` + `solang@v0.1.10` + `substrate-contracts-node@v0.8.0`

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

cargo install \
  contracts-node \
  --git https://github.com/paritytech/substrate-contracts-node.git \
  --tag v0.8.0 \
  --force \
  --locked
```

#### Compiling 💥 FAILS

``` bash
solang --target substrate -o ./artifacts/ ./contracts/mocks/RMRKCoreMock.sol
```

#### Running `substrate-contracts-node`

```bash
substrate-contracts-node --dev --ws-port 9944
```

#### Instantiating a `RMRKCoreMock` contract

```bash
wasm=$(jq -r .source.wasm ./artifacts/RMRKCoreMock.contract)
hash=$(jq -r .source.hash ./artifacts/RMRKCoreMock.contract)
value=1000000000000
gas_limit=1000000000000
storage_deposit_limit=1000000000000
ctor_selector=$(jq -r .spec.constructors[0].selector ./artifacts/RMRKCoreMock.contract)
data="${ctor_selector}30524d524b436f72654d6f636b10524d5243204372656174757265"
salt=0x0000000000000000000000000000000000000000000000000000000000000000

printf "%s null" $wasm > /tmp/rmrk.params

npx --yes @polkadot/api-cli@beta \
  --ws ws://localhost:9944 \
  --seed //Alice \
  --params /tmp/rmrk.params \
  tx.contracts.uploadCode

printf \
  "%d %d %d %s %s %s" \
  $value \
  $gas_limit \
  $storage_deposit_limit \
  $hash \
  $data \
  $salt \
> /tmp/rmrk.params

npx @polkadot/api-cli@beta \
  --ws ws://localhost:9944 \
  --seed //Alice \
  --params /tmp/rmrk.params \
  tx.contracts.instantiate

# TODO: mint some creatures
```

<details>
  <summary>How to easily generate the constructor payload <code>0x30524d524b436f72654d6f636b10524d5243204372656174757265</code></summary>

```rust
fn main() {
    println!(
        "0x{}{}{}",
        hex::encode(parity_scale_codec::Encode::encode("RMRKCoreMock")),
        hex::encode(parity_scale_codec::Encode::encode("RMRC")),
        hex::encode(parity_scale_codec::Encode::encode("Creature"))
    );
}
```
</details>