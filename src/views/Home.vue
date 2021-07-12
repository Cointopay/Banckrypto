<template>
  <div class="my-4">
    <div class="flex align-items justify-between">
      <h1 class="text-lg font-bold">Wallets & Addresses</h1>
      <button :disabled="isLoading" @click="generateNextAddress()"
              class="w-full inline-flex items-center justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-blue-600 text-base font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:w-auto sm:text-sm">
        <svg class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg"
             fill="none" viewBox="0 0 24 24" v-if="isLoading">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
        <span v-if="!isLoading">Create Next Address</span>
      </button>
    </div>
    <div class="overflow-x-auto border-gray-200 sm:rounded-lg mt-4">
      <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
        <tr>
          <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Address</th>
          <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Balance&nbsp;({{ currency }})
          </th>
          <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Total Received&nbsp;({{ currency }})
          </th>
          <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Actions</th>
        </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
        <tr v-for="(wallet) in wallets" :key="wallet.address">
          <td class="px-4 py-2 whitespace-nowrap">
            <router-link tag="a" :to="{ name: 'address', params: {address: wallet.address} }"
                         class="text-blue-700 hover:text-blue-900 hover:underline">
              {{ wallet.address }}
            </router-link>
            <span v-if="wallet.type"
                  class="ml-4 inline-flex items-center justify-center px-2 py-1 text-xs font-bold leading-none text-blue-700 bg-blue-100 rounded">
              {{ wallet.type }}
            </span>
          </td>
          <td class="px-4 py-2 whitespace-nowrap">
            {{ wallet.balance + ' ' + currency }}
          </td>
          <td class="px-4 py-2 whitespace-nowrap">
            {{ wallet.totalReceived + ' ' + currency }}
          </td>
          <td class="px-4 py-2 whitespace-nowrap">
            <a href="javascript:void(0)" class="hover:text-blue-700" @click="getPrivateKey(wallet.address)">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24"
                   stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M15 7a2 2 0 012 2m4 0a6 6 0 01-7.743 5.743L11 17H9v2H7v2H4a1 1 0 01-1-1v-2.586a1 1 0 01.293-.707l5.964-5.964A6 6 0 1121 9z"/>
              </svg>
            </a>
          </td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
  <verify-password></verify-password>
  <show-private-key></show-private-key>
</template>

<script>
import {mapGetters} from "vuex"
import VerifyPassword from "@/components/modals/VerifyPassword"
import ShowPrivateKey from "@/components/modals/ShowPrivateKey"
import axios from "axios"
import {SAVE_WALLETS} from "@/store/keys";

export default {
  name: "Dashboard",
  components: {ShowPrivateKey, VerifyPassword},
  data() {
    return {
      currency: process.env.VUE_APP_CURRENCY,
      baseUrl: process.env.VUE_APP_WALLET_URL,
      isLoading: false
    }
  },
  computed: {
    ...mapGetters([
      'walletId',
      'token',
      'wallets'
    ])
  },
  methods: {
    getPrivateKey(address) {
      this.emitter.emit('verify-password', {
        address: address
      })
    },
    generateNextAddress() {
      this.isLoading = true
      axios.post(`${this.baseUrl}`, JSON.stringify({
        method: 'getnewaddress',
        account: this.walletId
      }), {
        headers: {
          Authorization: 'Basic ' + this.token
        }
      }).then(response => {
        if (response.data.error === null) {
          const wallets = [...this.wallets]
          wallets.push({
            address: response.data.result,
            balance: 0,
            totalReceived: 0,
            type: ""
          })
          this.$store.commit(SAVE_WALLETS, wallets)
          this.isLoading = false
        } else {
          this.isLoading = false
          this.$toast.error(response.data.error)
        }
      }).catch(error => {
        this.isLoading = false
        this.$toast.error(error.response.data.error)
      })
    }
  }
}
</script>

<style scoped></style>