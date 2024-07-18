<template>
  <div id="app">
    <nav>
      <div class="left">
        <button @click="showForm = true">发布主题</button>
      </div>
      <div class="middle">
        <input type="text" placeholder="Search..." v-model="searchQuery" />
      </div>
      <div class="right">
        <button @click="connectWallet">连接钱包</button>
      </div>
    </nav>

    <!-- Use CustomModal instead of modal -->
    <CustomModal v-if="showForm" :visible="showForm" @close="showForm = false">
      <template v-slot:header>
        <h3>发布主题</h3>
      </template>
      <template v-slot:body>
        <input type="text" v-model="newCard.imgUrl" placeholder="Image URL" />
        <input type="text" v-model="newCard.url" placeholder="URL" />
        <input type="text" v-model="newCard.description" placeholder="Description" />
        <input type="text" v-model="newCard.author" placeholder="Author" />
      </template>
      <template v-slot:footer>
        <button @click="addProfileCard">Submit</button>
        <button @click="showForm = false">Close</button>
      </template>
    </CustomModal>

    <CustomModal v-if="showCardModal" :visible="showCardModal" @close="showCardModal = false">
      <template v-slot:header>
        <h3>Card Details</h3>
      </template>
      <template v-slot:body>
        <div class="card-img" :style="{ backgroundImage: 'url(' + selectedCard.imgUrl + ')' }"></div>
        <div class="card-content">
          <p><strong>Description:</strong> {{ selectedCard.description }}</p>
          <p><strong>URL:</strong> <a :href="selectedCard.url">{{ selectedCard.url }}</a></p>
          <p><strong>Author:</strong> {{ selectedCard.author }}</p>
          <p><strong>Views:</strong> {{ selectedCard.views }}</p>
        </div>
      </template>
      <template v-slot:footer>
        <button @click="showCardModal = false">Close</button>
      </template>
    </CustomModal>

    <main>
      <div class="cards-container">
        <div
          class="card"
          v-for="(card, index) in filteredProfileCards"
          :key="index"
          @click="openCardModal(index)"
        >
          <div class="card-img" :style="{ backgroundImage: 'url(' + card.imgUrl + ')' }"></div>
          <div class="card-content">
            <p><strong>Description:</strong> {{ truncateText(card.description, 100) }}</p>
            <p><strong>URL:</strong> <a :href="card.url" @click.stop>{{ card.url }}</a></p>
            <p><strong>Author:</strong> {{ card.author }}</p>
            <p><strong>Views:</strong> {{ card.views }}</p>
            <button class="donate-button">Donate</button>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import CustomModal from './CustomModal.vue';
import { ref } from 'vue';
import { createWeb3Modal, defaultWagmiConfig } from '@web3modal/wagmi/vue';
import { mainnet, arbitrum } from 'viem/chains';
import { reconnect } from '@wagmi/core';

// 1. Define constants
const projectId = '83ff5c8d3d124b29fcf8a12feb7f4118';

// 2. Create wagmiConfig
const metadata = {
  name: 'Web3Modal',
  description: 'Web3Modal Example',
  url: '', // origin must match your domain & subdomain
  icons: ['']
};

const chains = [mainnet, arbitrum];
const config = defaultWagmiConfig({
  chains,
  projectId,
  metadata,
  // Add wagmiOptions here if needed
});

// Reconnect if previously connected
reconnect(config);

// 3. Create modal
const web3Modal = createWeb3Modal({
  wagmiConfig: config,
  projectId,
  enableAnalytics: false, // Optional - defaults to your Cloud configuration
  enableOnramp: false // Optional - false as default
});

// Reactive variable to manage connection state
const isConnected = ref(false);

// Function to handle wallet connection
const connectWallet = async () => {
  try {
    await web3Modal.open();
    isConnected.value = true; // Update connection state
  } catch (error) {
    console.error('Failed to connect wallet:', error);
  }
};

export default {
  components: {
    CustomModal,
  },
  data() {
    return {
      profileCards: JSON.parse(localStorage.getItem('profileCards')) || [],
      searchQuery: '',
      showForm: false,
      showCardModal: false,
      selectedCard: null,
      newCard: {
        imgUrl: '',
        url: '',
        description: '',
        author: '',
        views: 0
      },
      isAuthenticated: false,
    };
  },
  methods: {
    addProfileCard() {
      if (this.newCard.imgUrl && this.newCard.url && this.newCard.description && this.newCard.author) {
        this.profileCards.push({ ...this.newCard });
        localStorage.setItem('profileCards', JSON.stringify(this.profileCards));
        this.newCard = { imgUrl: '', url: '', description: '', author: '', views: 0 };
        this.showForm = false;
      } else {
        alert('Please fill in all fields');
      }
    },
    openCardModal(index) {
      this.selectedCard = this.profileCards[index];
      this.selectedCard.views += 1;
      localStorage.setItem('profileCards', JSON.stringify(this.profileCards));
      this.showCardModal = true;
    },
    truncateText(text, maxLength) {
      return text.length > maxLength ? text.substring(0, maxLength) + '...' : text;
    },
    connectWallet // Ensure connectWallet is part of methods
  },
  computed: {
    filteredProfileCards() {
      return this.profileCards.filter((card) =>
        card.description.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    }
  },
  created() {
    this.isAuthenticated = localStorage.getItem('isAuthenticated') === 'true';
  }
};
</script>

<style>
body {
  margin: 0;
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
#app {
  background: 'bg.jpeg';
  min-height: 100vh;
  font-size: 17px;
  font-family: Wawati SC;
}
nav {
  background-color: white;
  padding: 3em;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
nav .left {
  flex: 1;
  font-size: 27px;
}
nav .middle {
  flex: 2;
  display: flex;
  justify-content: center;
}
nav .right {
  flex: 1;
  display: flex;
  font-size: 27px;
  justify-content: flex-end;
  align-items: center;
}
nav input[type="text"] {
  width: 80%;
  padding: 0.5em;
  font-size: 1em;
}
nav button {
  background: none;
  border: none;
  font-weight: bold;
  font-size: 1em;
  cursor: pointer;
  margin-left: 1em;
}
.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin-right: 10px;
}
main {
  padding: 1em;
  background: url('/src/assets/bg2.jpeg');
}
.cards-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1em;
  justify-content: center;
}
.card {
  background: white;
  color: black;
  width: calc(33% - 1em);
  display: flex;
  flex-direction: column;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0.7, 0.7, 0.7, 0.7);
  margin-bottom: 2em;
  cursor: pointer;
}
.card-img {
  background: black;
  height: 200px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}
.card-content {
  padding: 1em;
  flex-grow: 1;
}
.card-content a {
  color: blue;
  text-decoration: none;
}
.card-content a:hover {
  text-decoration: underline;
}
.donate-button {
  background: white;
  color: black;
  border: 2px solid black;
  padding: 0.5em;
  font-weight: bold;
  cursor: pointer;
  margin-top: 1em;
  align-self: flex-start;
}
</style>
