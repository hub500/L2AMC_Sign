<script setup>
// import { verifyMessage } from "@unisat/wallet-utils";
// import bitcore from "bitcore-lib";
import { ref } from "vue";
import { ethers, Signature, verifyMessage } from "ethers";
import { Buffer } from "buffer";

const address = ref("");
const address_eth = ref("");
const address_sol = ref("");
const pubkey = ref("");
const message = "hello";
const sign_result = ref("");
const sign_result_hex = ref("");
const sign_result_eth = ref("");
const sign_result_sol = ref("");

// function verifyMessage(publicKey, text, sig) {
//   const message = new bitcore.Message(text);

//   var signature = bitcore.crypto.Signature.fromCompact(
//     Buffer.from(sig, "base64")
//   );
//   var hash = message.magicHash();

//   // recover the public key
//   var ecdsa = new bitcore.crypto.ECDSA();
//   ecdsa.hashbuf = hash;
//   ecdsa.sig = signature;

//   const pubkeyInSig = ecdsa.toPublicKey();

//   const pubkeyInSigString = new bitcore.PublicKey(
//     Object.assign({}, pubkeyInSig.toObject(), { compressed: true })
//   ).toString();
//   if (pubkeyInSigString != publicKey) {
//     return false;
//   }

//   return bitcore.crypto.ECDSA.verify(hash, signature, pubkeyInSig);
// }

async function unisat_connect() {
  try {
    let accounts = await window.unisat.requestAccounts();
    console.log('connect success', accounts);
    address.value = accounts[0];
  } catch (e) {
    console.log('connect failed');
  }
}

async function unisat_publickey() {
  try {
    let res = await window.unisat.getPublicKey();
    console.log(res)
    pubkey.value = res;
  } catch (e) {
    console.log(e);
  }
}

// https://github.com/unisat-wallet/wallet-utils/blob/master/src/utils.ts
async function unisat_sign() {
  try {
    let res = await window.unisat.signMessage(message);
    if (!res) {
      return
    }
    let res_hex = Buffer.from(res, "base64").toString("hex");
    console.log(res);
    console.log(res_hex);
    sign_result.value = res;
    sign_result_hex.value = "0x" + res_hex;
  } catch (e) {
    console.log("[ERR]signMessage", e);
  }
  if (!sign_result.value) {
    return
  }

  try {
    let sig = Signature.from(sign_result_hex.value);
    console.log(sig.v, sig.r, sig.s);
    console.log("v", ethers.getBigInt(sig.v));
    console.log("r", ethers.getBigInt(sig.r));
    console.log("s", ethers.getBigInt(sig.s));
  } catch (e) {
    console.log("[ERR]Signature", e);
  }

  // const result = verifyMessage(pubkey.value, message, sign_result.value);
  // console.log(result);
}

// eth
async function eth_signer() {
  let provider = new ethers.BrowserProvider(window.ethereum);
  let signer = await provider.getSigner();
  console.log(signer);
  return signer;
}

async function eth_connect() {
  let accounts = await window.ethereum.request({
    method: "eth_requestAccounts",
    jsonrpc: "2.0",
  });
  console.log(accounts);
  address_eth.value = ethers.getAddress(accounts[0]);
}

// https://docs.metamask.io/wallet/reference/eth_getencryptionpublickey/
async function eth_publickey() {
  let pubkey = await window.ethereum.request({
    "method": "eth_getEncryptionPublicKey",
    "params": [
      address_eth.value
    ]
  });
  console.log(pubkey);
}

// https://eips.ethereum.org/EIPS/eip-191
async function eth_sign() {
  let signer = await eth_signer();
  let sign_result = await signer.signMessage(message);
  console.log(sign_result);

  let sig = Signature.from(sign_result);
  console.log(sig.v, sig.r, sig.s);
  console.log("v", ethers.getBigInt(sig.v));
  console.log("r", ethers.getBigInt(sig.r));
  console.log("s", ethers.getBigInt(sig.s));

  let sig_addr = verifyMessage(message, sign_result);
  console.log(sig_addr, address_eth.value, ethers.getAddress(sig_addr) == address_eth.value);

  sign_result_eth.value = sign_result;
}

const getProvider = () => {
  if ('phantom' in window) {
    const provider = window.phantom?.solana;

    if (provider?.isPhantom) {
      return provider;
    }
  }

  window.open('https://phantom.app/', '_blank');
};

async function sol_connect() {
  try {
    const wallet = await getProvider().request({ method: "connect" });
    console.log(wallet);
    console.log(wallet.publicKey.toString());
    console.log(wallet.publicKey.toBase58());
    address_sol.value = wallet.publicKey.toString();
  } catch (err) {
    console.log(err);
    // { code: 4001, message: 'User rejected the request.' }
  }
}

async function sol_publickey() {
  await sol_connect();
}

async function sol_sign() {
  const provider = getProvider(); // see "Detecting the Provider"
  const encodedMessage = new TextEncoder().encode(message);
  const signedMessage = await provider.signMessage(encodedMessage, "utf8");
  console.log(signedMessage);
  console.log(signedMessage.publicKey.toString());
  console.log(signedMessage.signature.toString("hex"));

  try {
    const sign_result = signedMessage.signature.toString("hex");
    let sig = Signature.from("0x" + sign_result);
    console.log(sig.v, sig.r, sig.s);
    console.log("v", ethers.getBigInt(sig.v));
    console.log("r", ethers.getBigInt(sig.r));
    console.log("s", ethers.getBigInt(sig.s));
  } catch (e) {
    console.log("[ERR]Signature", e);
  }
}
</script>

<template>
  <div>
    <a href="https://chrome.google.com/webstore/detail/unisat/ppbibelpcjmhbdihakflkdcoccbgbkpo" target="_blank">
      UniSat Wallet Install
    </a>
  </div>
  <div>
    <a href="https://chromewebstore.google.com/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn" target="_blank">
      Metamask Wallet Install
    </a>
  </div>
  <div>
    unisat wallet <strong>{{ address }}</strong>
  </div>
  <div>
    unisat public key <strong>{{ pubkey }}</strong>
  </div>
  <div>
    <button @click="unisat_connect">unisat connect</button>
    <button @click="unisat_publickey">unisat public key</button>
    <button @click="unisat_sign">unisat sign message</button>
  </div>
  <div>{{ sign_result }}</div>
  <div>{{ sign_result_hex }}</div>

  <hr />

  <div>
    eth wallet <strong>{{ address_eth }}</strong>
  </div>
  <div>
    <button @click="eth_connect">eth connect</button>
    <button @click="eth_publickey">eth public key</button>
    <button @click="eth_sign">eth sign message</button>
  </div>

  <hr />

  solana wallet <strong>{{ address_sol }}</strong>

  <div>
    <button @click="sol_connect">sol connect</button>
    <button @click="sol_publickey">sol public key</button>
    <button @click="sol_sign">sol sign message</button>
  </div>
  <div>{{ sign_result_eth }}</div>
</template>
