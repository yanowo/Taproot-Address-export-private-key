<script>
import  {
  mnemonicToSeed,
} from '@scure/bip39';

import {
  HDKey,
} from '@scure/bip32';

import * as btc from 'micro-btc-signer'

export default {
  data() {
    return {
      mnemonic: '',
      path: "m/86'/0'/0'/0/0",
      privateKey: '',
      address: '',
      url:'https://github.com/yanowo/Taproot-Address-export-private-key',
      privateKeysInput: "",
      addresses: [],
    }
  },
  methods: {
    copy(text) {
      this.$copyText(text);
      alert(`Copied ${text}`);
    },
	getAddressesFromPrivateKeys() {
	  const privateKeys = this.privateKeysInput.split("\n").filter((key) => key.trim() !== "");

	  const addresses = privateKeys.map((privateKey, index) => {
		try {
		  const keyPair = Buffer.from(privateKey, "hex");
		  const address = btc.getAddress('tr', privateKey);
		  return { index, address, privateKey };
		} catch (error) {
		  console.error(`Error processing private key at index ${index}:`, error);
		  return { index, address: 'Error', privateKey, error };
		}
	  });

	  this.addresses = addresses;
	},
    async getPrivateKey() {
      try {
        const masterseed = await mnemonicToSeed(this.mnemonic);

        const network_version = {
          mainnet: {
            private: 0x04b2430c,
            public: 0x04b24746,
          },
        };

        const hdkey = HDKey.fromMasterSeed(masterseed, network_version.mainnet);

        const receive_path = this.path;

        const receive_node = hdkey.derive(receive_path);

        const address = btc.getAddress('tr', receive_node.privateKey);
        this.privateKey = Buffer.from(receive_node.privKeyBytes).toString('hex');
        this.address = address;        
      } catch (e) {
        alert(e.message)
      }
    },
    async deriveAddressFromPrivateKey() {
      if (!this.privateKey) {
      alert("Please enter private key");
      return;
      }

      try {
      const privateKeyBuffer = Buffer.from(this.privateKey, "hex");
      if (privateKeyBuffer.length !== 32) {
        throw new Error("Invalid private key length");
      }
      const address = btc.getAddress('tr', privateKeyBuffer);
      this.address = address;
      } catch (error) {
      alert("Error: " + error.message);
      }
    }
  }
}

</script>

<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    This project is open source hosted at
    <p>
      <button type="button">https://github.com/yanowo/Taproot-Address-export-private-key</button>
      <button @click="copy(url)">Copy</button>
    </p>
    <p>
      Everyone should download the source code and review it and run it locally
    </p>
    <p>
      <b style="color: red">Please run this at your own risk if you run this on a network, if you are very stubborn or not technical,
        please disconnect your network and exit this page after getting the private key</b>
    </p>
  </div>
<h2>Batch Private Keys Input:</h2>
<textarea v-model="privateKeysInput" rows="10" cols="50"></textarea><br>
<button @click="getAddressesFromPrivateKeys">Get Addresses</button>


<h2>Generated Addresses:</h2>
<ul>
  <li v-for="item in addresses" :key="item.index" class="address-container">
    <div class="address-text">{{ item.address }}</div>

      <button class="copy-button" @click="copy(item.address)">Copy Address</button>
      <button class="copy-button" @click="copy(item.privateKey)">Copy Private Key</button>

  </li>
</ul>

  <p>
  Private Key：
  </p>
  <p><input class="mnemonic-input" v-model="privateKey" /></p><button @click="deriveAddressFromPrivateKey">Get public address</button>
  <br><br><br><br>
  <p>
    Your mnemonic:
  </p>
  <p><input class="mnemonic-input" v-model="mnemonic" /></p>
  <p>
    Path:
  </p>
  <p>
    <input class="mnemonic-input" v-model="path" />
  </p>
  <button @click="getPrivateKey">Get private key</button>
  <template v-if="address || privateKey">
    <p>
      Your address
    </p>
    <p>
      {{ address }}
      <button @click="copy(address)">Copy</button>
    </p>

    <p>
      Your privateKey
    </p>
    <p>
      {{ privateKey }}
      <button @click="copy(privateKey)">Copy</button>
    </p>
  </template>

  <p class="read-the-docs">If this works for you and you want to thank this is the donation address:
    bc1p8srf8extm6xjuf7lasxcjqtf2mseh3uhunu0vtwtttlvwq4axlhs26jx2a</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
.mnemonic-input {
  width: 300px;
  height: 30px;
  margin: 4px 16px;
}
</style>
