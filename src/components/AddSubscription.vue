<template>
  <v-card class="rounded-t-lg rounded-b-0">
    <v-card-title>
      <v-row no-gutters>
          Subscription Add
        <v-spacer/>
        <v-btn color="red" icon @click="close()">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-row>
    </v-card-title>
    <v-card-text>
      <v-select :items="plans" item-text="name"
              item-value="id"  v-model="current_subscription_id" outlined label="Select the plan: " class="rounded-lg"></v-select>
    </v-card-text>

    <v-card-actions>
      <v-spacer/>
      <v-btn @click="editUser" dark large color="primary">
        <v-icon class="mr-2">mdi-check</v-icon>
        Save</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
import {authClient} from "@/plugins/http";

export default {
  name: "AddSubscription",
  data: () => ({
    editSheet: false,
    plans: [],
    editData: {},
    current_subscription_id: 0,
    gender:[
      {text:'Male',value:'male'},
      {text:'Female',value:'female'}
    ]
  }),
  methods:{
    close(){
      this.$emit('close')
    },
    getPlans(){
      this.loading = true;
      authClient.get('/common/subscriptions/get-all').then(res=>{
        this.plans = res.data;  
        this.loading = false;
      }).catch(err=>{
        this.loading = false;
      })
    },
    editUser(){
      authClient.post('/subscriptions/adminSubscribe', {"userId": this.data.id, "subscriptionId": this.current_subscription_id }).then(r=>{
        console.log(r);
        this.close();
      });
    }
  },
  mounted() {
    this.getPlans()
    this.editData = this.data;
    this.editData.nic = this.editData.nic.toUpperCase();
    console.log(this.editData);
  },
  props: {
    data: {
      type: Object,
      required: true
    }
  },
}
</script>

<style scoped>

</style>
