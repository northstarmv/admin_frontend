<template>
    <v-container>
      <v-row no-gutters>
        <v-text-field v-model="search" outlined rounded label="Search Thrapies..."></v-text-field>
        <v-btn @click="addUserDialog = true; editTherapy = null; isEdit = false" class="ml-2" rounded height="56" elevation="0" dark>
          <v-icon>mdi-plus</v-icon>
          Hold User
        </v-btn>
      </v-row>
  
      <v-card outlined :loading="loading">
        <v-row no-gutters> 
            <v-col
               class="pt-3"
                cols="3"
              >
              <template >
                <div class="mr-0 mr-sm-3">
                  <v-menu
                    ref="menu"
                    v-model="menu"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    min-width="auto"
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                       @click:clear="date = null; getTherapies()"
                        clearable
                        v-model="date"
                        label="Date"
                        prepend-icon="mdi-calendar"
                        readonly
                        outlined
                        v-bind="attrs"
                        v-on="on"
                      ></v-text-field>
                    </template>
                    <v-date-picker
                      v-model="date"
                      :active-picker.sync="activePicker"
                      :max="(new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substring(0, 10)"
                      min="1950-01-01"
                      @change="save"
                    ></v-date-picker>
                  </v-menu>
                </div>
              </template>
            </v-col>
            
            
          </v-row>
          <v-data-table
            :loading="loading"
            :search="search"
            :headers="headers"
            :items="meetings">
          <template v-slot:item.reason="{ item }">
            <v-btn outlined rounded @click="descriptionViewDialog = true; selectedDescription = item.reason">
              View
            </v-btn>
          </template>

          <template v-slot:item.additional="{ item }">
            <v-btn outlined rounded @click="descriptionViewDialog = true; selectedDescription = item.additional">
              View
            </v-btn>
          </template>
  
  
  
          <!-- <template v-slot:item.actions="{ item }">
            <v-btn @click="isEdit = true; editStory = item; addStoryDialog = true" icon>
              <v-icon>mdi-pencil</v-icon>
            </v-btn>
            <v-btn @click="showDeleteConfirmation(item, deleteStory)" icon>
              <v-icon>mdi-delete</v-icon>
            </v-btn>
            <v-btn @click="statusChange(item.id, item.status === 1? 2 : 1)" icon>
              <v-icon :color=" item.status === 2 ? 'success': 'warning'"> {{ item.status === 1 ? 'mdi-cancel' : 'mdi-check' }}</v-icon>
            </v-btn>
          </template> -->
        </v-data-table>
      </v-card>

      <v-dialog max-width="768" persistent v-if="descriptionViewDialog" v-model="descriptionViewDialog">
        <v-card>
          <v-card-title>
            <v-row no-gutters>
              <span class="headline">Description</span>
              <v-spacer/>
              <v-btn @click="descriptionViewDialog = false; selectedDescription=''" icon color="red">
                <v-icon>mdi-close</v-icon>
              </v-btn>
            </v-row>
          </v-card-title>
          <v-divider class="mb-4"/>
          <v-card-text>
            {{ selectedDescription }}
          </v-card-text>
        </v-card>
      </v-dialog>
  
  
      <v-dialog max-width="768" persistent v-if="addUserDialog" v-model="addUserDialog">
        <HoldUserDialog :therapy='editTherapy' :edit="isEdit" @close="closeDialogs"/>
      </v-dialog>
    </v-container>
  </template>
  
  <script>
  import {authClient} from "@/plugins/http";
  import HoldUserDialog from "@/components/HoldUserDialog.vue";
  
  export default {
    name: "TherapyMeeting",
    components: {HoldUserDialog},
    data: () => ({
      descriptionViewDialog:false,
      activePicker: null,
      date: null,
      menu: false,
      search: '',
      loading: true,
      meetings: [],
      headers: [
        {text: "Client", value: "name"},
        {text: "Email", value: "email"},
        {text: "Date", value: "apt_date"},
        {text: "Start Time", value: "start_time"},
        {text: "End Time", value: "end_time"},
        {text: "Description", value: "reason"},
        {text: "Additional", value: "additional"},
        // {text: "Actions", value: "actions"},
      ],
      addUserDialog: false,
      editTherapy: null,
      isEdit: false
    }),
    methods: {
      editUserDetails(user) {
        this.editTherapy = user;
        this.isEdit = true;
        this.addUserDialog = true;
      },
      save (date) {
          this.$refs.menu.save(date)
          this.getTherapies()
      },
      closeDialogs(){
        this.getTherapies();
        this.addUserDialog = false;
        this.editTherapy = null;
        this.isEdit = false;
      },
      getTherapies() {
        authClient.post('/meeting/therapy-meetings',{"apt_date":this.date}).then((response) => {
          let data = response.data[0]
          if(data.status){
            this.meetings = data.data.result;
            this.domain = data.data.domain;
          }else{
            this.loading = false;
            this.$toast.error({
              title: 'error',
              message: data.message,
          });
          }
          this.loading = false;
        }).catch((error) => {
          console.log(error);
          this.loading = false;
        });
      },
      deleteTherapy(item) {
          authClient.post('/therapy/delete',{
            userId: item.user_id,
            therapy_id: item.therapy_Id
          }).then((response) => {
            let data = response.data[0]
            if(data.status){
              this.$toast.success({
                title: 'Ok',
                message: data.message,
              });
              this.getTherapies()
            }else{
              this.loading = false;
              this.$toast.error({
                title: 'error',
                message: data.message,
            });
          }
            
          }).catch((error) => {
            console.log(error);
          });
      },
      showDeleteConfirmation(item, callback) {
        this.$toast.question({
          timeout: 10000, // The time in milliseconds the toast will be displayed
          close: false, // Whether to show the close button
          overlay: true, // Whether to display an overlay behind the toast
          toastOnce: true, // Whether to show the toast only once
          id: 'deleteToast',
          zindex: 999,
          title: 'Confirmation',
          message: 'Are you sure you want to delete?',
          position: 'center',
          buttons: [
            ['<button><b>YES</b></button>', function (instance, toast) {
              callback(item)
              instance.hide({ transitionOut: 'fadeOut' }, toast, 'button');
            }, true],
            ['<button>NO</button>', function (instance, toast) {
              instance.hide({ transitionOut: 'fadeOut' }, toast, 'button');
            }],
          ]
        });
      },
    },
    mounted() {
      this.getTherapies();
    }
  }
  </script>
  
  <style scoped>
  
  </style>
  