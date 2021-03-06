<template>
  <div class="descriptionContainer">
    <div class="secondary-background text-white" v-if="this.$store.state.web3.networkId != this.$store.state.requiredNetwork">
      Please switch to Ethereum {{ this.$store.state.netName }}
    </div>
    <b-card
      class="header-card"
      overlay
      :img-src="background"
      img-alt="EthKids"
      img-height="300vw"
      text-variant="white"
    >
      <b-card-body class="top-header-row">
        <b-row align-h="between">
          <b-col md="2" class="d-none d-md-block text-left">
            <router-link to="/">

            </router-link>
          </b-col>
          <b-col>
            <div class="secondary" id="rewards">
              <span class="balance-container biggerText bolderText">
                <a target="_blank" v-bind:href="getTokenLink">
                  {{ myBalance }} {{ this.$store.state.tokenSym }}
                </a>
              </span>
              <b-button
                size="default"
                class="confirmBtn shadow-lg"
                v-show="this.$store.state.tokenMyBalance > 0"
                v-on:click="sellBack()">
                Sell
              </b-button>
            </div>
            <b-tooltip target="rewards">
              Value: ≈ {{ parseFloat(this.$store.state.tokenMyETHReturn).toFixed(2) }} ΞTH
              <br>
              Community vault: <a target="_blank" v-bind:href="getBondingVaultAddressLink">{{ this.$store.state.bondingVaultBalance }} ΞTH</a>
            </b-tooltip>
          </b-col>
          <b-col cols="4">
            <div id="nav" class="bolderText text-right pr-4 secondary">
              <router-link to="/">
                <span class="ml-md-3 mr-md-3">Home</span>
              </router-link>
              |
              <router-link to="/about">
                <span class="ml-md-3 mr-md-3">
                    About
                </span>
              </router-link>
              <div v-if="confirming" class="secondary ml-md-3 mr-md-3">
                Confirming
                <b-spinner small variant="danger" label="Spinning"></b-spinner>
              </div>
            </div>
          </b-col>
          <b-col lg="1"/>
        </b-row>

        <b-row align-h="between">
          <b-col md="10" style="padding-top: 70px">
            <span class="niceFont secondary header-title-label">
              EthKids
            </span>
            <br>
            <span class="header-main-label">Charitable giving</span>
          </b-col>
        </b-row>

      </b-card-body>
    </b-card>
  </div>
</template>

<script>
import State from "@/mixins/State";

export default {
  name: "HeaderPlate",
  mixins: [State],
  components: {},
  data() {
    return {
      background: require('@/assets/header-new.png'),
      confirming: false,
      myBalance: 0,
    }
  },
  beforeCreate() {
    const self = this;
    EventBus.subscribe('TX_CONFIRMING', () => {
      self.confirming = true;
    });
    EventBus.subscribe('TX_CONFIRMED', () => {
      self.confirming = false;
    });
  },
  computed: {
    getTokenLink() {
      return `${this.$store.state.etherscan}/token/${this.$store.state.tokenAddress}`;
    },
    getBondingVaultAddressLink() {
      return `${this.$store.state.etherscan}/address/${this.$store.state.bondingVaultAddress}`;
    },
  },
  mounted() {
    const self = this;
    this.$store.subscribe((mutation) => {
      if (mutation.type === 'registerToken') {
        self.loadMyTokenBalance();
      }
      if (mutation.type === 'registerWeb3Instance') {
        //happens initially or upon changing account
        if (self.$store.state.tokenInstance) {
          self.loadMyTokenBalance();
        }
      }
      if (mutation.type === 'financialChange') {
        self.reloadFinancialState();
      }
      if (mutation.type === 'registerBondingVault') {
        self.loadBondingVault();
      }
    });
  },
  methods: {
    sellBack() {
      EventBus.publish('OPEN_SELL');
    },
    reloadFinancialState() {
      this.loadMyTokenBalance();
      this.loadBondingVault();
    },
    loadMyTokenBalance() {
      let tokenInstance = this.$store.state.tokenInstance();
      let self = this;
      tokenInstance.methods.balanceOf(this.$store.state.web3.coinbase).call().then((tokenBalance) => {
        const balanceInETH = self.fromWei(tokenBalance);
        self.$store.commit('registerTokenMyBalance', balanceInETH);
        self.myBalance = Math.floor(Number(balanceInETH) * 100) / 100;
        self.loadMyReturn();
      });

    },
    loadMyReturn() {
      if (this.myBalance > 0 && this.$store.state.bondingVaultInstance) {
        const amountWei = this.toWei(this.myBalance.toString(), 'ether');
        this.$store.state.bondingVaultInstance().methods.calculateReturn(amountWei)
          .call({from: this.$store.state.web3.coinbase}).then((result) => {
          this.$store.commit('registerTokenMyETHReturn', this.fromWei(result));
        }).catch((e) => {
          throw e;
        });
      } else {
        this.$store.commit('registerTokenMyETHReturn', "0");
      }
    },
    loadBondingVault() {
      let bondingVaultContract = this.$store.state.bondingVaultInstance();
      this.xWeb3().web3Instance.eth.getBalance(bondingVaultContract.options.address, (err, bondingVaultBalance) => {
        let balanceEth = this.fromWei(bondingVaultBalance);
        this.$store.commit('registerBondingVaultBalance', parseFloat(balanceEth).toFixed(2));
      });
    },
  }
}
</script>

<style scoped>
.descriptionContainer {
  margin-bottom: 10px;
}

.header-card {
  background-color: white;
  border: none;
}

.card-img {
  object-fit: cover !important;
  object-position: 0 50%;
}

.top-header-row {
  padding: 0;
}

.header-title-label {
  font-size: 56px;
}

.balance-container {
  padding: 10px;
  background-color: white;
  border: white;
  border-radius: 20px;
  color: #40494e;
}

.balance-container a {
  color: #40494e;
}

.header-main-label {
  font-size: 28px;
  font-weight: 200;
  color: white;
}

</style>
