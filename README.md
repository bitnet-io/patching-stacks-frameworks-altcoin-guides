# patching-stacks-frameworks-altcoin-guides
dh weinberg (c4pt000) Bitnet IO founder and main developer

for each project

stacks-blockchain-api
stacks-core-node
stack-explorer
stack.js 

npm install --force
modules must always be patched manually......hunt them down and change them with your chainparams.cpp from your coin
hex is always 0x and decimal is always the bare number

nano node_modules/bitcoinjs-lib/src/networks.js
```
25 = B and 22 is the p2sh hash in decimal
adjust to your network
https://github.com/bitnet-io/bitnet-core/blob/main/src/chainparams.cpp#L176-L182

25 = 0x19 in hex
22 = 0x16 in hex
158 = 0x9e in hex

https://github.com/bitnet-io/bitcoinjs-lib/blob/master/src/networks.js#L5-L14

p2pkh
p2sh
wif
'bc'
```

nano node_modules/c32check/lib/address.js
```
25 = B and 22 is the p2sh hash in decimal
adjust to your network
https://github.com/bitnet-io/bitnet-core/blob/main/src/chainparams.cpp#L176-L182
25 = 0x19 in hex
22 = 0x16 in hex
158 = 0x9e in hex

export const versions = {
  mainnet: {
    p2pkh: 25, // 'P'
    p2sh: 22, // 'M'
  },
  testnet: {
    p2pkh: 26, // 'T'
    p2sh: 21, // 'N'
  },
};

// address conversion : bitcoin to stacks
const ADDR_BITCOIN_TO_STACKS: Record<number, number> = {};
ADDR_BITCOIN_TO_STACKS[25] = versions.mainnet.p2pkh;         // keep this as 0 when using c32check for stacks.js node_modules
ADDR_BITCOIN_TO_STACKS[22] = versions.mainnet.p2sh;          // keep this as 5 when using c32check for stacks.js node_modules
ADDR_BITCOIN_TO_STACKS[111] = versions.testnet.p2pkh;
ADDR_BITCOIN_TO_STACKS[196] = versions.testnet.p2sh;

// address conversion : stacks to bitcoin
const ADDR_STACKS_TO_BITCOIN: Record<number, number> = {};
ADDR_STACKS_TO_BITCOIN[versions.mainnet.p2pkh] = 25;         // keep this as 0 when using c32check for stacks.js node_modules 
ADDR_STACKS_TO_BITCOIN[versions.mainnet.p2sh] = 22;          // keep this as 5 when using c32check for stacks.js node_modules
ADDR_STACKS_TO_BITCOIN[versions.testnet.p2pkh] = 111;
ADDR_STACKS_TO_BITCOIN[versions.testnet.p2sh] = 196;

let prefix = bitcoinVersion.toString(16);
  if (prefix.length === 1) {
    prefix = `25${prefix}`;
  }


```
nano node_modules/c32check/lib/encoding.js
```
https://github.com/bitnet-io/c32check/blob/master/src/encoding.ts#L55-L58

here \u0000 = 0x00 in hex for my network its \u0019 where 0x19 is 25 in decimal for legacy B

  const zeroPrefix = new TextDecoder().decode(hexToBytes(inputHex)).match(/^\u0000*/);
  const numLeadingZeroBytesInHex = zeroPrefix ? zeroPrefix[0].length : 0;


  const zeroPrefix = new TextDecoder().decode(hexToBytes(inputHex)).match(/^\u0019*/);
  const numLeadingZeroBytesInHex = zeroPrefix ? zeroPrefix[0].length : 0;



```
nano node_modules/@stacks/transactions/dist/constants.js and dist/esm/constants.js (both files)

https://github.com/bitnet-io/stacks.js/blob/main/packages/transactions/src/constants.ts#L160-L189

```
SerializeP2PKH = 0x00,
  /** `MultiSigHashMode` — hash160(multisig-redeem-script), same as bitcoin's multisig p2sh */
  SerializeP2SH = 0x01,


my network

SerializeP2PKH = 0x19,
  /** `MultiSigHashMode` — hash160(multisig-redeem-script), same as bitcoin's multisig p2sh */
  SerializeP2SH = 0x16,


export enum AddressVersion {
  /** `P` — A single-sig address for mainnet (starting with `SP`) */
  MainnetSingleSig = 22,
  /** `M` — A multi-sig address for mainnet (starting with `SM`) */
  MainnetMultiSig = 20,
  /** `T` — A single-sig address for testnet (starting with `ST`) */


my network

export enum AddressVersion {
  /** `P` — A single-sig address for mainnet (starting with `SP`) */
  MainnetSingleSig = 25,
  /** `M` — A multi-sig address for mainnet (starting with `SM`) */
  MainnetMultiSig = 22,
  /** `T` — A single-sig address for testnet (starting with `ST`) */


```


FOR LEATHER WALLET PATCHING HERE
```
npm install --force
modify the node_modules exactly the same way for
bitcoinjs-lib
bip32/src/bip32.js
c32check/address.js
c32check/encoding/js
@stacks/transactions/dist/constants.js
@stacks/transactions/dist/esm/constants.js
```

in the leather source code not the node_modules

nano shared/crypto/bitcoin/bitcoin.network.ts
```

my network bitnet-io

const bitcoinMainnet: BtcSignerNetwork = {
  bech32: 'bit',
  pubKeyHash: 0x19,
  scriptHash: 0x16,
  wif: 0x9e,
};
```
stacks address generator depends on @stacks/transactions/dist/constants.js and dist/esm/constants.js
also c32check/address.js and c32check/encoding.js
also bitcoinjs-lib/src/network.js

these files in leather wallet control and connect to your api
NANO ALL OF THEM AND SET YOUR CONSTANTS
```
shared/constants.ts:export const BITCOIN_API_BASE_URL_MAINNET = 'https://bitexplorer.io/api';
app/features/add-network/add-network.tsx:        setBitcoinUrl('https://bitexplorer.io/api');
app/features/add-network/add-network.tsx:        setBitcoinUrl('https://bitexplorer.io/signet/api');
app/features/add-network/add-network.tsx:        setBitcoinUrl('https://bitexplorer.io/testnet/api');
app/query/bitcoin/bitcoin-client.ts:    const resp = await axios.get(`https://bitexplorer.io/api/address/${address}/txs`);
app/query/bitcoin/ordinals/brc20/brc20-tokens.query.ts:			// const res = await axios.get(`http://bitexplorer.io:1717/fsck`);
app/query/bitcoin/ordinals/brc20/brc20-tokens.query.ts:			//  const res = await axios.get(`http://bitexplorer.io:1717/bit1p6r4nsxdnh4slwr6w9j6m0h64jnelldyh548spqy97e0gv5x4lxyqpvr9a8`);

┌─[root@nwstrtrj01]─[/home/c4pt/opt/leather-orig-6.26.1-STX-BITNET-lastworking-03-05-2024/src]
└──╼ #grep -ri "bitnft.io"
shared/constants.ts:export const HIRO_API_BASE_URL_MAINNET = 'https://bitnft.io';
shared/constants.ts:export const HIRO_INSCRIPTIONS_API_URL = 'https://bitnft.io/ordinals/v1/inscriptions';
app/features/add-network/add-network.tsx:        setStacksUrl('https://bitnft.io');
app/features/stacks-transaction-request/principal-value.tsx:      onClick={() => openInNewTab(`https://explorer.bitnft.io/address/${address}?chain=${mode}`)}
app/query/bitcoin/bitcoin-client.ts:      `https://bitnft.io/api/address/${address}/utxo`
app/query/bitcoin/ordinals/inscription.hooks.ts:  return `https://bitnft.io/inscription/${id}`;
app/query/bitcoin/ordinals/brc20/brc20-tokens.query.ts: const res = await axios.get(`https://bitnft.io/ordinals/v1/bit-20/tokens/${ticker}`);
app/query/bitcoin/ordinals/brc20/brc20-tokens.query.ts:  const res = await axios.get(`https://bitnft.io/ordinals/v1/bit-20/balances/${address}`);
app/common/utils.ts:  return `https://explorer.bitnft.io/txid/${txid}?chain=${mode}${suffix}`;

ALSO

grep -ri "'BTC'" and change those values to your symbol

also these file fetch prices from price apis
┌─[root@nwstrtrj01]─[/home/c4pt/opt/leather-orig-6.26.1-STX-BITNET-lastworking-03-05-2024/src]
└──╼ #ls -l app/query/common/market-data/vendors/
total 12
-rw-rw-r-- 1 root root 739 Mar  5 15:34 binance-market-data.query.ts
-rw-rw-r-- 1 root root 819 Mar  5 15:34 coincap-market-data.query.ts
-rw-rw-r-- 1 root root 871 Mar  5 15:34 coingecko-market-data.query.ts

```





