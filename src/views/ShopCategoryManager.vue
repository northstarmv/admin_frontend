<template>
  <v-container>
    <v-row no-gutters>
      <v-text-field v-model="search" outlined rounded label="Search Categories..."></v-text-field>
      <v-btn @click="addNew = true;isEdit=false;" rounded dark color="black" height="56" elevation="0" class="ml-2">
        <v-icon class="mr-2">mdi-plus</v-icon>
        Add New
      </v-btn>
    </v-row>

    <v-card outlined :loading="loading">
      <v-data-table
          :search="search"
          :headers="headers"
          :items="categories">
        <!--        <template v-slot:item.image_path="{ item }">
                  <v-avatar class="ma-2">
                    <v-img :src="$store.state.s3ResourcesBaseUrl + item.image_path"></v-img>
                  </v-avatar>
                </template>-->
        <template v-slot:item.actions="{ item }">
          <v-icon @click="editCategory(item)" class="mr-2" color="primary">mdi-pencil</v-icon>
          <v-icon @click="showDeleteConfirmation(item, deleteCategory)" color="error">mdi-delete</v-icon>
        </template>
      </v-data-table>
    </v-card>

    <v-bottom-sheet v-if="addNew" v-model="addNew" inset persistent>
      <v-card class="rounded-t-lg rounded-b-0">
        <v-card-title>
          <v-row no-gutters>
            {{ isEdit ? 'Edit' : 'Add' }} Category
            <v-spacer/>
            <v-btn color="red" icon @click="close()">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </v-row>
        </v-card-title>

        <v-card-text>
          <v-form ref="form" v-model="valid">
            <v-text-field v-model="data.name" outlined label="Name" :rules="[rules.required]"></v-text-field>
          </v-form>
        </v-card-text>

        <v-card-actions style="background: #1976d2">
          <v-spacer/>
          <v-btn :loading="loading" @click="addNewCategory" large color="white">
            <v-icon class="mr-2">mdi-check</v-icon>
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-bottom-sheet>
  </v-container>
</template>

<script>
import {authClient} from "@/plugins/http";

export default {
  name: "ShopCategoryManager",
  data: () => ({
    rules: {
      required: value => !!value || 'Required.',
    },
    data: {
      id: null,
      name: '',
    },
    valid: false,
    image: null,
    search: '',
    loading: true,
    addNew: false,
    isEdit: false,
    headers: [
      {text: "ID", value: "id"},
      {text: "Name", value: "name"},
      {text: "Actions", value: "actions", sortable: false},
    ],
    categories: [],
  }),
  methods: {
    deleteCategory(item) {
          this.loading = true
          authClient.post('/ecom/categories/actions/delete',{
            id: item.id
          }).then((response) => {
            this.loading = false
            let data = response.data[0]
            if(data.status){
              this.$toast.success({
                title: 'Ok',
                message: data.message,
              });
              this.getCategories()
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
    editCategory(item) {
      this.data = item;
      this.isEdit = true;
      this.addNew = true;
    },
    addNewCategory() {
      this.$refs.form.validate();
      if (this.valid) {
        this.loading = true;
        authClient.post('/ecom/categories/actions/upsert', this.data).then(res => {
          let response_data = res.data[0]
              if(response_data.status){
                this.loading = false;
                this.close();
                this.data = {
                  id: null,
                  name: '',
                };
                this.$toast.success({
                  title: 'Ok',
                  message: response_data.message,
              });
              }else{
                this.loading = false;
                this.$toast.error({
                  title: 'error',
                  message: response_data.message[0],
              });
              }
        }).catch(err => {
          this.loading = false;
          this.$toast.error({
                  title: 'error',
                  message: 'error',
              });
        });
      }
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
    close() {
      this.data = {
        id: null,
        name: '',
      };
      this.addNew = false;
      this.getCategories();
    },
    getCategories() {
      this.loading = true;
      this.data = {
        id: null,
        name: '',
      };
      authClient.get('/ecom/categories/actions/list', {
        'search_key': this.search,
      }).then(res => {
        let data = res.data[0]
          if(data.status){
            this.categories = data.data.result;
            this.loading = false;
          }else{
            this.loading = false;
            this.$toast.error({
              title: 'error',
              message: data.message,
          });
          }
      }).catch(err => {
        console.log(err);
        this.loading = false;
      })
    }
  },
  mounted() {
    this.getCategories();
  }
}
</script>

<style scoped>

</style>
