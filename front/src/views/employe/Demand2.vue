<template>
  <v-app
        :style="{  background: '#F5F5F5' }"
      >
        <SideBar/>
        <v-main>
    <v-container class="px-1 mt-n2" >
      <v-row>
        <v-col >
           
        </v-col>
        <v-col>
           
                <NavBar />
          
           
        </v-col>
    </v-row>

   </v-container>

  
    <v-container class="px-6 mt-n2" fluid>
      <v-data-table
       :headers="headers"
       :items="requests"
       height="500"
>
  
<template v-slot:top>
    <v-toolbar
      flat
      :color="'rgb(232, 252, 255)'"
    >
      <v-toolbar-title>Requests</v-toolbar-title>
      <v-divider
        class="mx-4"
        inset
        vertical
      ></v-divider>
      <v-spacer></v-spacer>
      <v-dialog
          v-model="dialog"
          max-width="700px"
        >
          <template v-slot:activator="{ props }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="props"
            >
              New Request
            </v-btn>
          </template>
          <v-card>
            <v-card-title>  
              <span class="text-h5">New demand</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-text-field
                      v-model="editedItem.fullname"
                      label="Fullname"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-text-field
                      v-model="editedItem.department"
                      label="department"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-text-field
                      v-model="editedItem.position"
                      label="position"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-text-field
                      v-model="editedItem.problem"
                      label="problem"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-textarea
                      v-model="editedItem.description"
                      label="description"
                      rows="3"
                      row-height="15"
                    ></v-textarea>
                </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue-darken-1"
                variant="text"
                @click="close"
              >
                Cancel
              </v-btn>
              <v-btn
                color="blue-darken-1"
                variant="text"
                @click="save"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog
          v-model="dialogMore"
          max-width="500px"
        >
          
          <v-card>
            <v-card-title>
              <span class="text-h5">Answer</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                    <v-col
                    cols="12"
                    sm="6"
                    md="4"
                    >
                        <p class="text-h6">
                          Answer
                        </p>
                    </v-col>
                   </v-row>
                   <v-row>
                    <v-col
                    cols="12"
                    sm="12"
                    >
                        <p class="text-body-1">
                         {{ editedItem.answer }}
                        </p>
                    </v-col>
                   </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue-darken-1"
                variant="text"
                @click="close"
              >
                Close
              </v-btn>
              
            </v-card-actions>
          </v-card>
        </v-dialog>
    </v-toolbar>
  </template>

  <template v-slot:no-data>
    <v-btn
      color="primary"
      @click="initialize"
    >
      Reset
    </v-btn>
  </template>
  <template v-slot:item.progress="{ item }">
      <div class="text-center ">
        <v-chip
          :color="getColor(item.progress)"
          :text="getText(item.progress)"
          class="text-uppercase"
          label
          size="small"
        ></v-chip>
      </div>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
        size="small"
        class="me-2"
        @click="seeMore(item)"
      >
        mdi-eye
      </v-icon>
    
    </template>
</v-data-table>
    </v-container>
 



   </v-main>

        
      </v-app>
</template>

<script setup>
  import NavBar from "@/components/employe/NavBar.vue";
  import SideBar from "@/components/employe/SideBare.vue";
import axios from 'axios';
</script>
<script>
export default {
  data: () => ({
    loading:false,
    dialog: false,
    dialogMore: false,
    headers: [
      {
        title: 'Fullname',
        align: 'start',
        sortable: false,
        key: 'fullname',
      },
      { title: 'Department', key: 'department' },
      { title: 'Position', key: 'position' },
      { title: 'Problem', key: 'problem', },
      { title: 'Actions', key: 'actions', sortable: false ,align: 'center'},
      { title:  'Progress', key:'progress',align: 'center'},
      
     
    ],
    requests: [],
    editedIndex:-1,
    editedItem: {
     fullname:'',
     department:'',
     position:'',
     problem:'',
     progress:'',
     answer:'',
     description:''
    },
    defaultItem: {
     fullname:'',
     department:'',
     position:'',
     problem:'',
     progress:'',
     answer:'',
     description:''
    },
    
  }),



  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogMore (val) {
        val || this.close()
      },
    
  },

  created () {
    this.fetchData()
  },

  methods: {
    fetchData() {
      const dataRaw = localStorage.getItem('data');
        const  data = JSON.parse(dataRaw);
      this.loading=true
  axios.get(`http://localhost:8000/reclamation/only/${data.id}`,)
    .then(response => {
      this.requests = response.data;
    })
    .catch(error => {
      console.error('Error fetching data:', error);
    }).finally(()=>{
      this.loading=false;
    })
},

    editItem (item) {
      this.editedIndex = this.requests.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },
    seeMore(item) {
      this.editedIndex = this.requests.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogMore= true
    },

    close () {
      this.dialogMore=false
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

   
    async save () {
        const dataRaw = localStorage.getItem('data');
        const  data = JSON.parse(dataRaw);
        this.editedItem.progress = 'y'
        this.editedItem.id=data.id
        this.editedItem.answer=''
        try{
           const response = await axios.post('http://localhost:8000/reclamation/new', this.editedItem)
           if(response.data.state==='S'){
            this.requests.push(this.editedItem)
           }
           console.log('request success:', response);
          }catch(error){
            console.error('requests fail:', error);
          // Handle error, show error message, etc.
          }
          
        this.close()
      },

    getColor(progress){
      switch(progress){
          case 'R': return'red'
          case 'A': return'green'
          default : return 'yellow'
      }
    },
    getText(progress){
      switch(progress){
          case 'R': return'Refused'
          case 'A': return'Accepted'
          default : return 'Waiting'
      }
    }

  },
}
</script>

<style scoped></style>
