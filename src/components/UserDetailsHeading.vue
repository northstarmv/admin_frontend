<template>
  <div>
    <v-row align="center" no-gutters>
      <v-img class="rounded-xl" max-height="128" min-width="128" min-height="128" max-width="128" :src=" $store.state.s3BaseUrl + data.avatar_url"></v-img>
      <v-col class="ml-8">
        <span class="text-h4 font-weight-bold">{{ data.name }}</span><br>
        <v-chip v-if="(data.role !=='doctor') && (data.role !=='gym')" dark
                :color="checkActiveSub ? '#ffa200':''">
          {{ checkActiveSub  ? 'PRO Member':'Member' }}
        </v-chip>
        <v-chip v-if="data.role ==='doctor'" dark :color="data.doctor.online ? 'green':''">{{ data.doctor.online  ? 'Online':'Offline' }}</v-chip>
      </v-col>



      <v-spacer/>

      <v-btn class="mr-2" v-if="data.role !== 'gym'" @click="addProUser = true" elevation="0" dark rounded color="#ffa200">
        <v-icon class="mr-2">mdi-plus</v-icon>Pro
      </v-btn>

      <v-btn v-if="data.role !== 'gym'" @click="editSheet = true" elevation="0" dark rounded color="#ffa200">
        <v-icon class="mr-2">mdi-pencil</v-icon>Edit
      </v-btn>

      <v-btn v-if="data.role === 'gym'" @click="editGymSheet = true" elevation="0" dark rounded color="#ffa200">
        <v-icon class="mr-2">mdi-pencil</v-icon>Edit
      </v-btn>

      

    </v-row>
    <v-bottom-sheet inset v-model="addProUser">
      <AddSubscription :data="data" @close="editSheet = false;$router.back()"/>
    </v-bottom-sheet>

    <v-bottom-sheet inset v-model="editSheet">
      <UserEditBasicData :data="data" @close="editSheet = false;$router.back()"/>
    </v-bottom-sheet>
    

    <v-bottom-sheet scrollable inset v-model="editGymSheet">
      <GymUserEditBasicData :gym="gym" @soft-close="editGymSheet = false;" @close="close"/>
    </v-bottom-sheet>
  </div>
</template>

<script>
import UserEditBasicData from "@/components/UserEditBasicData";
import GymUserEditBasicData from "@/components/GymEditBasicData";
import AddSubscription from "@/components/AddSubscription";
export default {
  name: "UserDetailsHeading",
  components: {GymUserEditBasicData, UserEditBasicData, AddSubscription},
  computed : {
    checkActiveSub(){
      if(this.subscription == null){
          return false;
        } else {
          return this.subscription.is_active
        }
    }
  },
  data: () => ({
    hasActiveSub: false,
    editSheet: false,
    addProUser: false,
    editGymSheet: false,
  }),
  props: {
    data: {
      type: Object,
      required: true
    },
    gym: {
      type: Object
    },
    subscription: {
      type: Object,
      required: true
    },
  },
  methods:{
    close(){
      this.editGymSheet = false;
      this.$emit('close');
    }
  },
  mounted() {
  }
}
</script>

<style scoped>

</style>
