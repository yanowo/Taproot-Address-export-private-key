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
      path: "m/86'/0'/0'/0",
      privateKey: '',
      address: '',
      url:'ttps://github.com/yanowo/Taproot-Address-export-private-key'
    }
  },
  methods: {
    copy(text) {
      this.$copyText(text);
      alert(`Copied ${text}`);
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
    bc1plsjrevcge5w0pryecqpsr3g4gyjrdn4wgdgy6654yqtjkepx26jqvzd5jq</p>
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
