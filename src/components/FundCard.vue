<template>
  <b-col lg="5">
    <donate-modal
      v-bind:name='name'
      v-bind:title='title'
    />
    <pass-charity-modal
      v-bind:name='name'
      v-bind:title='title'
    />
    <div class="fundContainer">
      <div>
        <h2 class="mt-0">
          <a v-bind:href="url" target="_blank">
            {{ title }}
          </a>
        </h2>
        <ul class="social list-inline">
          <li v-if="twitter" class="list-inline-item">
            <a v-bind:href="twitter" target="_blank" class="icon">
              <font-awesome-icon size="lg" :icon="['fab', 'twitter']"/>
            </a>
          </li>
          <li v-if="vk" class="list-inline-item">
            <a v-bind:href="vk" target="_blank" class="icon">
              <font-awesome-icon size="lg" :icon="['fab', 'vk']"/>
            </a>
          </li>
          <li v-if="fb" class="list-inline-item">
            <a v-bind:href="fb" target="_blank" class="icon">
              <font-awesome-icon size="lg" :icon="['fab', 'facebook-f']"/>
            </a>
          </li>
          <li v-if="youtube" class="list-inline-item">
            <a v-bind:href="youtube" target="_blank" class="icon">
              <font-awesome-icon size="lg" :icon="['fab', 'youtube']"/>
            </a>
          </li>
          <li v-if="instagram" class="list-inline-item">
            <a v-bind:href="instagram" target="_blank" class="icon">
              <font-awesome-icon size="lg" :icon="['fab', 'instagram']"/>
            </a>
          </li>
          <li v-if="linkedin" class="list-inline-item">
            <a v-bind:href="linkedin" target="_blank" class="icon">
              <font-awesome-icon size="lg" :icon="['fab', 'linkedin']"/>
            </a>
          </li>
        </ul>

      </div>

      <div class="fundDescription">
        <slot name="description">
        </slot>
      </div>

      <b-carousel
        v-if="images"
        id="carousel-fade"
        style="text-shadow: 0px 0px 2px #000"
        fade
        controls
        indicators
        img-width="300"
        img-height="300"
      >
        <b-carousel-slide v-for="image in images" v-bind:key="image.src"
                          v-bind:img-src="image.src"
        ></b-carousel-slide>
      </b-carousel>


      <FundFinancialState
        v-bind:name='name'
      />
      <div class="row justify-content-center">
        <div class="actions">
          <b-button
            size="lg"
            class="confirmBtn shadow-lg"
            v-on:click="donate()">
            Donate
          </b-button>
        </div>
        <div v-if="isAdmin">
          <b-button
            size="lg"
            class="confirmBtn shadow-lg"
            v-on:click="passCharity()">
            Pass to charity
          </b-button>
        </div>
      </div>
      <div class="row justify-content-around" style="margin-top: 20px">
        <b-button variant="outline-info" @click="toggleDonations">Last {{ donations }} donations</b-button>
        <b-button variant="outline-info" @click="toggleTransfers">Last {{ transfers }} transfers</b-button>
      </div>
      <div style="margin: 20px">
        <b-collapse v-model="donationsVisible" class="mt-2">
          <b-card>
            <DonationsTrail
              v-bind:name='name'
            />
          </b-card>
        </b-collapse>
        <b-collapse v-model="transfersVisible" class="mt-2">
          <b-card>
            <TransfersTrail
              v-bind:name='name'
            />
          </b-card>
        </b-collapse>
      </div>
    </div>

  </b-col>

</template>

<script>
import EventBus from '@/utils/event-bus';
import FundFinancialState from '@/components/FundFinancialState'
import DonationsTrail from '@/components/DonationsTrail'
import DonateModal from '@/components/DonateModal';
import PassCharityModal from '@/components/PassCharityModal';
import TransfersTrail from "./TransfersTrail";
import State from "@/mixins/State";

export default {
  name: 'FundCard',
  mixins: [State],
  props: {
    name: String,
    title: String,
    url: String,
    twitter: String,
    vk: String,
    fb: String,
    youtube: String,
    instagram: String,
    linkedin: String,
    images: Array,
  },
  data() {
    return {
      isAdmin: false,
      donationsVisible: false,
      transfersVisible: false,
      donations: 0,
      transfers: 0,
    };
  },
  mounted() {
    const self = this;
    this.$store.subscribe((mutation) => {
      if (mutation.type == 'registerCommunity' && mutation.payload.name === this.name) {
        mutation.payload.contract().methods.isWhitelistAdmin(self.$store.state.web3.coinbase).call().then((isSigner) => {
          self.isAdmin = isSigner;
        });
      }
      if (mutation.type == 'registerCommunityDonation' && mutation.payload.name === this.name) {
        self.loadDonations();
      }
      if (mutation.type == 'registerCommunityTransfer' && mutation.payload.name === this.name) {
        self.loadTransfers();
      }
    });
    this.loadDonations();
    this.loadTransfers();
  },
  methods: {
    loadDonations() {
      if (this.community(this.name)) {
        this.donations = this.community(this.name).communityDonations ? this.community(this.name).communityDonations.length : 0;
      }
    },
    loadTransfers() {
      if (this.community(this.name)) {
        this.transfers = this.community(this.name).communityTransfers ? this.community(this.name).communityTransfers.length : 0;
      }
    },
    toggleDonations() {
      this.donationsVisible = !this.donationsVisible;
      if (this.donationsVisible) {
        this.transfersVisible = false;
      }
    },
    toggleTransfers() {
      this.transfersVisible = !this.transfersVisible;
      if (this.transfersVisible) {
        this.donationsVisible = false;
      }
    },
    donate() {
      if (this.$store.state.readOnly) {
        EventBus.publish('SHOW_WEB3_MODAL', {
          callback: () => {
            EventBus.publish('OPEN_DONATE', {community: this.name});
          }
        })
      } else {
        EventBus.publish('OPEN_DONATE', {community: this.name});
      }
    },
    passCharity() {
      if (this.$store.state.readOnly) {
        EventBus.publish('SHOW_WEB3_MODAL', {
          callback: () => {
            EventBus.publish('PASS_CHARITY', {community: this.name});
          }
        })
      } else {
        EventBus.publish('PASS_CHARITY', {community: this.name});
      }
    },
  },
  components: {
    TransfersTrail,
    FundFinancialState,
    DonationsTrail,
    DonateModal,
    PassCharityModal,
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.fundContainer {
  padding: 10px;
  margin-bottom: 20px;
  border: 1px dashed #b8b8b8;
  border-radius: 10px;
}

.social {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

ul {
  text-align: left;
}

ul li {
  display: inline-block;
}

.fundDescription {
  text-align: left;
  text-justify: inter-word;
}

</style>
