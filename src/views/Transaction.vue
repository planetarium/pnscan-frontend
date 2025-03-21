<template>
  <v-container fluid class="px-0">
    <div class="page-wide-wrap detail-page px-md-12">
      <h1 class="page-title px-4 px-md-0">Transaction Details</h1>
      <v-divider class="my-2"></v-divider>

      <v-card class="mt-4" outlined>
        <v-progress-linear indeterminate height="2" v-if="loading"></v-progress-linear>
        <v-tabs>
          <v-tab>Overview</v-tab>
        </v-tabs>
        <v-divider></v-divider>
        <v-card-text class="pa-0">
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Transaction Hash:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">
              {{id}}  <copy-btn :text="tx.id" icon x-small><v-icon small>mdi-content-copy</v-icon></copy-btn>
            </v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Status:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading && (tx.id || tx.status)">
              <div v-if="statusLoading"><v-progress-circular indeterminate class="mr-2" size="12" width="2"></v-progress-circular></div>
              <div v-else>
                <v-chip label small color="success" v-if="tx.status == 'SUCCESS'">SUCCESS</v-chip>
                <v-chip label small color="error" v-else-if="tx.status == 'FAILED'" @click="refreshTxStatus">FAILED</v-chip>
                <v-chip label small color="secondary" v-else-if="tx.status == 'INVALID' || tx.status == null" @click="refreshTxStatus">NOT FOUND</v-chip>
                <v-chip label small color="warning" v-else @click="refreshTxStatus">{{tx.status}}</v-chip>
              </div>
            </v-col>
            <v-col cols="12" sm="9" class="item-value" v-else-if="!loading && !tx.id">
              <v-chip label small color="warning"><v-progress-circular indeterminate class="mr-2" size="12" width="2"></v-progress-circular>WAITING</v-chip>
            </v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Block:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading && tx.id">
              <router-link :to="{name: 'block', params:{index:tx.blockIndex}}">{{tx.blockIndex}}</router-link>
              <v-chip small label class="ml-2" color="#eee"><strong class="mr-1">{{latestBlockIndex - tx.blockIndex + 1}}</strong> Block confirmations</v-chip>
            </v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Timestamp:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading && tx.id">
              <v-icon small>mdi-clock-outline</v-icon> {{moment(tx.timestamp).fromNow()}} <span style="color:#777">({{ moment(tx.timestamp) }}</span>
            </v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Signer:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">
              <router-link :to="{name: 'account', params:{address: tx.signer}}">{{tx.signer}}</router-link>
            </v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Updated Addresses:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">
              <span v-for="addr in tx.updatedAddresses">
                <router-link :to="{name: 'account', params:{address:addr}}">{{addr}}</router-link>
                <br></span>
            </v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Signature:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">{{tx.signature}}</v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Public Key:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">{{tx.publicKey}}</v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Nonce:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">{{tx.nonce}}</v-col>
          </v-row>
        </v-card-text>
      </v-card>

      <v-card class="mt-4" outlined>
        <v-progress-linear indeterminate height="2" v-if="loading"></v-progress-linear>
        <v-tabs>
          <v-tab>Action Data</v-tab>
        </v-tabs>
        <v-divider></v-divider>
        <v-card-text class="py-12" v-if="!tx.actions || tx.actions.length == 0">
          No Action Data
        </v-card-text>
        <v-card-text class="pa-0" v-for="{inspection} in tx.actions" v-if="inspection">
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Type:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">
              {{inspection['typeId']}}
            </v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Avatar:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">
              <router-link :to="{name: 'avatar', params:{address: inspection['avatarAddress']}}" v-if="inspection['avatarAddress']">{{inspection['avatarAddress']}}</router-link>
            </v-col>
          </v-row>
          <v-row class="info-item ma-0">
            <v-col cols="12" sm="3" class="item-title">Id:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">
              {{inspection['id']}}
            </v-col>
          </v-row>
          <v-row class="info-item ma-0 data-row" v-for="k in Object.keys(inspection)" v-if="['txIdSeq', 'actionCount', 'typeId', 'timestamp', 'avatarAddress', 'id'].indexOf(k) == -1">
            <v-col cols="12" sm="3" class="item-title">{{k}}:</v-col>
            <v-col cols="12" sm="9" class="item-value" v-if="!loading">
              {{inspection[k]}}
            </v-col>
          </v-row>
          <action-descriptor :type="inspection['typeId']" :data="inspection"></action-descriptor>
        </v-card-text>
      </v-card>
    </div>

  </v-container>
</template>

<script>
import api from "../api"
import {mapGetters} from "vuex"
import BlockTable from "@/components/BlockTable";
import TransactionTable from "@/components/TransactionTable";
import CopyBtn from "@/components/btn/CopyBtn";
import ActionDescriptor from "@/components/ui/ActionDescriptor";
import Vue from "vue"
export default {
    name: 'Transaction',
    components: {ActionDescriptor, CopyBtn, TransactionTable, BlockTable},
    mixins: [],
    data() {
        return {
            loading: false,
            statusLoading: false,
            id: this.$route.params.id,
            tx: {}
        }
    },
    computed: {
      ...mapGetters('Block', ['latestBlockIndex'])
    },
    async created() {
        this.$watch('$route.params.id', async () => {
            this.id = this.$route.params.id
            this.init()
        })
        this.init()
        this.$store.dispatch('Sheet/init')
    },
    methods: {
        async init() {
            await this.loadTx()
            if (!this.tx.id) {
                this.watchBlock()
            }
        },
        async loadTx() {
            if (this.loading) return
            this.loading = true
            let tx = await this.$store.dispatch('Block/loadTransaction', this.id)
            if (tx) {
              this.tx = tx
            }
            if (!this.tx.status || this.tx.status == 'STAGING' || this.tx.status == 'INVALID') {
                this.refreshTxStatus()
            }
            this.loading = false
        },
        async refreshTxStatus() {
            try {
                this.statusLoading = true
                let status = await this.$store.dispatch('Block/loadTransactionStatus', this.id)
                if (status) {
                    Vue.set(this.tx, 'status', status)
                }
            } catch(e) {}
            this.statusLoading = false
        },
        watchBlock() {
            let cancelWatch = this.$watch('latestBlockIndex', async () => {
                await this.loadTx()
                if (this.tx.id) {
                    cancelWatch()
                }
                console.log(this.latestBlockIndex)
            })
        },
    }
}
</script>

<style scoped lang="scss">
.data-row {
  background-color: #fafafa;
  .item-title {
    font-weight: 400;
    color: #333;
    padding-left:32px;
  }
  .item-value {
    color:#333;
  }
}
</style>
