<template>
  <q-page class="flex column flex-center">
    <div class="flex row items-center q-mb-lg" >
      <div class="q-mr-lg">
        <q-btn label="ADD USER DETAILS" color="primary" @click="addUser" />
      </div>
    
    <div class="q-ml-lg">
      <q-btn label="DELETE ALL USER DETAILS" color="primary" @click="deleteUser" />
    </div>
    </div>
    <q-dialog v-model="isEdit">
      <q-card >
         <q-card-section class="row items-center" style="min-width: 350px;">
   <div class="text-h6">Add Details</div>
   
      <q-space />
        <q-btn icon="close"  class="row items-center" flat round dense  @click="clearAttribute"/>
         </q-card-section>
         <div class="q-pa-md" style="max-width: 400px">
       <q-form
      @submit="onSubmit"
      @reset="onReset"
      class="q-gutter-md"
    >
      <q-input
        filled
        v-model="name"
        label="Your name *"
        hint="Full Name"
        lazy-rules
        :rules="[ val => val && val.length > 0 || 'Please type something']"
      />

     <q-input v-model="age" filled type="date" hint="DOB" />
      <div>
        <q-btn label="Submit" type="submit" color="primary"/>
        <q-btn label="Reset" type="reset" color="primary" flat class="q-ml-sm" />
      </div>
       </q-form>
         </div>
       
      </q-card>
    </q-dialog>
    <q-table
      ref="tableRef"
      title="Treats"
      :rows="rows"
      :columns="columns"
      row-key="id"
      v-model:pagination="pagination"
      :loading="loading"
      :filter="filter"
      binary-state-sort
      @request="onRequest"
      selection="multiple"
      v-model:selected="selected"
    >
    
      <template v-slot:top-right>
        <q-btn
          color="primary"
          icon-right="delete_forever"
          no-caps
          @click="deleteSelected"
          class="q-mr-md"
        />
        <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </template>    
    </q-table>
    

  </q-page>
</template>

<script>
import { defineComponent,onMounted,ref } from 'vue'
import { useQuasar } from 'quasar'
const columns = [
  {
    name: 'desc',
    required: true,
    label: 'Dessert (100g serving)',
    align: 'left',
    field: row => row.name,
    format: val => `${val}`,
    sortable: true
  },
  { name: 'calories', align: 'center', label: 'Calories', field: 'calories', sortable: true },
  { name: 'fat', label: 'Fat (g)', field: 'fat', sortable: true },
  { name: 'carbs', label: 'Carbs (g)', field: 'carbs', sortable: true },
  { name: 'protein', label: 'Protein (g)', field: 'protein', sortable: true },
  { name: 'sodium', label: 'Sodium (mg)', field: 'sodium', sortable: true },
  { name: 'calcium', label: 'Calcium (%)', field: 'calcium', sortable: true, sort: (a, b) => parseInt(a, 10) - parseInt(b, 10) },
  { name: 'iron', label: 'Iron (%)', field: 'iron', sortable: true, sort: (a, b) => parseInt(a, 10) - parseInt(b, 10) }
]

const originalRows = [
  { id: 1, name: 'frozen Yogurt', calories: 159, fat: 6.0, carbs: 24, protein: 4.0, sodium: 87, calcium: '14%', iron: '1%' },
  { id: 2, name: 'ice cream sandwich', calories: 237, fat: 9.0, carbs: 37, protein: 4.3, sodium: 129, calcium: '8%', iron: '1%' },
  { id: 3, name: 'eclair', calories: 262, fat: 16.0, carbs: 23, protein: 6.0, sodium: 337, calcium: '6%', iron: '7%' },
  { id: 4, name: 'cupcake', calories: 305, fat: 3.7, carbs: 67, protein: 4.3, sodium: 413, calcium: '3%', iron: '8%' },
  { id: 5, name: 'gingerbread', calories: 356, fat: 16.0, carbs: 49, protein: 3.9, sodium: 327, calcium: '7%', iron: '16%' },
  { id: 6, name: 'jelly bean', calories: 375, fat: 0.0, carbs: 94, protein: 0.0, sodium: 50, calcium: '0%', iron: '0%' },
  { id: 7, name: 'lollipop', calories: 392, fat: 0.2, carbs: 98, protein: 0, sodium: 38, calcium: '0%', iron: '2%' },
  { id: 8, name: 'honeycomb', calories: 408, fat: 3.2, carbs: 87, protein: 6.5, sodium: 562, calcium: '0%', iron: '45%' },
  { id: 9, name: 'donut', calories: 452, fat: 25.0, carbs: 51, protein: 4.9, sodium: 326, calcium: '2%', iron: '22%' },
  { id: 10, name: 'kitKat', calories: 518, fat: 26.0, carbs: 65, protein: 7, sodium: 54, calcium: '12%', iron: '6%' },
  { id: 11, name: 'frozen Yogurt-1', calories: 159, fat: 6.0, carbs: 24, protein: 4.0, sodium: 87, calcium: '14%', iron: '1%' },
  { id: 12, name: 'ice cream sandwich-1', calories: 237, fat: 9.0, carbs: 37, protein: 4.3, sodium: 129, calcium: '8%', iron: '1%' },
  { id: 13, name: 'eclair-1', calories: 262, fat: 16.0, carbs: 23, protein: 6.0, sodium: 337, calcium: '6%', iron: '7%' },
  { id: 14, name: 'cupcake-1', calories: 305, fat: 3.7, carbs: 67, protein: 4.3, sodium: 413, calcium: '3%', iron: '8%' },
  { id: 15, name: 'gingerbread-1', calories: 356, fat: 16.0, carbs: 49, protein: 3.9, sodium: 327, calcium: '7%', iron: '16%' },
  { id: 16, name: 'jelly bean-1', calories: 375, fat: 0.0, carbs: 94, protein: 0.0, sodium: 50, calcium: '0%', iron: '0%' },
  { id: 17, name: 'lollipop-1', calories: 392, fat: 0.2, carbs: 98, protein: 0, sodium: 38, calcium: '0%', iron: '2%' },
  { id: 18, name: 'honeycomb-1', calories: 408, fat: 3.2, carbs: 87, protein: 6.5, sodium: 562, calcium: '0%', iron: '45%' },
  { id: 19, name: 'donut-1', calories: 452, fat: 25.0, carbs: 51, protein: 4.9, sodium: 326, calcium: '2%', iron: '22%' },
  { id: 20, name: 'kitKat-1', calories: 518, fat: 26.0, carbs: 65, protein: 7, sodium: 54, calcium: '12%', iron: '6%' },
  { id: 21, name: 'frozen Yogurt-2', calories: 159, fat: 6.0, carbs: 24, protein: 4.0, sodium: 87, calcium: '14%', iron: '1%' },
  { id: 22, name: 'ice cream sandwich-2', calories: 237, fat: 9.0, carbs: 37, protein: 4.3, sodium: 129, calcium: '8%', iron: '1%' },
  { id: 23, name: 'eclair-2', calories: 262, fat: 16.0, carbs: 23, protein: 6.0, sodium: 337, calcium: '6%', iron: '7%' },
  { id: 24, name: 'cupcake-2', calories: 305, fat: 3.7, carbs: 67, protein: 4.3, sodium: 413, calcium: '3%', iron: '8%' },
  { id: 25, name: 'gingerbread-2', calories: 356, fat: 16.0, carbs: 49, protein: 3.9, sodium: 327, calcium: '7%', iron: '16%' },
  { id: 26, name: 'jelly bean-2', calories: 375, fat: 0.0, carbs: 94, protein: 0.0, sodium: 50, calcium: '0%', iron: '0%' },
  { id: 27, name: 'lollipop-2', calories: 392, fat: 0.2, carbs: 98, protein: 0, sodium: 38, calcium: '0%', iron: '2%' },
  { id: 28, name: 'honeycomb-2', calories: 408, fat: 3.2, carbs: 87, protein: 6.5, sodium: 562, calcium: '0%', iron: '45%' },
  { id: 29, name: 'donut-2', calories: 452, fat: 25.0, carbs: 51, protein: 4.9, sodium: 326, calcium: '2%', iron: '22%' },
  { id: 30, name: 'kitKat-2', calories: 518, fat: 26.0, carbs: 65, protein: 7, sodium: 54, calcium: '12%', iron: '6%' },
  { id: 31, name: 'frozen Yogurt-3', calories: 159, fat: 6.0, carbs: 24, protein: 4.0, sodium: 87, calcium: '14%', iron: '1%' },
  { id: 32, name: 'ice cream sandwich-3', calories: 237, fat: 9.0, carbs: 37, protein: 4.3, sodium: 129, calcium: '8%', iron: '1%' },
  { id: 33, name: 'eclair-3', calories: 262, fat: 16.0, carbs: 23, protein: 6.0, sodium: 337, calcium: '6%', iron: '7%' },
  { id: 34, name: 'cupcake-3', calories: 305, fat: 3.7, carbs: 67, protein: 4.3, sodium: 413, calcium: '3%', iron: '8%' },
  { id: 35, name: 'gingerbread-3', calories: 356, fat: 16.0, carbs: 49, protein: 3.9, sodium: 327, calcium: '7%', iron: '16%' },
  { id: 36, name: 'jelly bean-3', calories: 375, fat: 0.0, carbs: 94, protein: 0.0, sodium: 50, calcium: '0%', iron: '0%' },
  { id: 37, name: 'lollipop-3', calories: 392, fat: 0.2, carbs: 98, protein: 0, sodium: 38, calcium: '0%', iron: '2%' },
  { id: 38, name: 'honeycomb-3', calories: 408, fat: 3.2, carbs: 87, protein: 6.5, sodium: 562, calcium: '0%', iron: '45%' },
  { id: 39, name: 'donut-3', calories: 452, fat: 25.0, carbs: 51, protein: 4.9, sodium: 326, calcium: '2%', iron: '22%' },
  { id: 40, name: 'kitKat-3', calories: 518, fat: 26.0, carbs: 65, protein: 7, sodium: 54, calcium: '12%', iron: '6%' }
]
export default defineComponent({
  name: 'IndexPage',
  setup () {
    const tableRef = ref()
    const rows = ref([])
    const filter = ref('')
    const loading = ref(false)
    const selected=ref([])
    const $q = useQuasar()
    const isEdit=ref(false)

    const name = ref(null)
    const age = ref(null)
    const accept = ref(false)
    const addUser=()=>{
      isEdit.value=true
    }
    const clearAttribute=()=>{
      isEdit.value=false
      name.value = null
        age.value = null
    }
    const pagination = ref({
      sortBy: 'desc',
      descending: false,
      page: 1,
      rowsPerPage: 5,
      rowsNumber: 10
    })
    const deleteUser=()=>{
      localStorage.clear()
       $q.notify({
            color: 'green-4',
            textColor: 'white',
            icon: 'cloud_done',
            message: 'All User Details Deleted Successfully'
          })
    }

      function deleteSelected(){  
      selected.value.filter(function(item){
        rows.value.splice(rows.value.indexOf(item), 1);
        return item;
      });
      selected.value = [];
    }
  
  
    function fetchFromServer (startRow, count, filter, sortBy, descending) {
      const data = filter
        ? originalRows.filter(row => row.name.includes(filter))
        : originalRows.slice()

      // handle sortBy
      if (sortBy) {
        const sortFn = sortBy === 'desc'
          ? (descending
              ? (a, b) => (a.name > b.name ? -1 : a.name < b.name ? 1 : 0)
              : (a, b) => (a.name > b.name ? 1 : a.name < b.name ? -1 : 0)
            )
          : (descending
              ? (a, b) => (parseFloat(b[ sortBy ]) - parseFloat(a[ sortBy ]))
              : (a, b) => (parseFloat(a[ sortBy ]) - parseFloat(b[ sortBy ]))
            )
        data.sort(sortFn)
      }

      return data.slice(startRow, startRow + count)
    }

    
    function getRowsNumberCount (filter) {
      if (!filter) {
        return originalRows.length
      }
      let count = 0
      originalRows.forEach(treat => {
        if (treat.name.includes(filter)) {
          ++count
        }
      })
      return count
    }

    function onRequest (props) {
      const { page, rowsPerPage, sortBy, descending } = props.pagination
      const filter = props.filter

      loading.value = true

    
      setTimeout(() => {
        // update rowsCount with appropriate value
        pagination.value.rowsNumber = getRowsNumberCount(filter)

        // get all rows if "All" (0) is selected
        const fetchCount = rowsPerPage === 0 ? pagination.value.rowsNumber : rowsPerPage

        // calculate starting row of data
        const startRow = (page - 1) * rowsPerPage

        // fetch data from "server"
        const returnedData = fetchFromServer(startRow, fetchCount, filter, sortBy, descending)

        // clear out existing data and add new
        rows.value.splice(0, rows.value.length, ...returnedData)

        // don't forget to update local pagination object
        pagination.value.page = page
        pagination.value.rowsPerPage = rowsPerPage
        pagination.value.sortBy = sortBy
        pagination.value.descending = descending

        // ...and turn of loading indicator
        loading.value = false
      }, 1500)
    }

    onMounted(() => {
      // get initial data from server (1st page)
      tableRef.value.requestServerInteraction()
    })

    return {
      tableRef,
      filter,
      loading,
      pagination,
      columns,
      rows,
      selected,
      isEdit,
      name,
      age,
      accept,
      deleteSelected,
      addUser,
      deleteUser,
      clearAttribute,
       onSubmit () {
         
         let myarray=[]
         myarray=JSON.parse(localStorage.getItem("myObject"))
         if(myarray==null){
           myarray=[]
         }
         let myObject={
           Name:name.value,
           DOB:age.value
         }
         if(myarray.length==0){
          myarray.push(myObject)
         }
         else {
           let i=myarray.length
           myarray[i]=myObject
         }
         
         localStorage.setItem("myObject", JSON.stringify(myarray))
          $q.notify({
            color: 'green-4',
            textColor: 'white',
            icon: 'cloud_done',
            message: 'Submitted'
          })
          
        clearAttribute()
      },

      onReset () {
        name.value = null
        age.value = null
        accept.value = false
      },
      //  prompt () {
      //   $q.dialog({
      //     title: 'Prompt',
      //     message: 'What is your name? (Minimum 3 characters)',
      //     prompt: {
      //       model: '',
      //       isValid: val => val.length > 2, // << here is the magic
      //       type: 'text' // optional
      //     },

      //     cancel: true,
      //     persistent: true
      //   }).onOk(data => {
      //     // console.log('>>>> OK, received', data)  
      // })
      // },

      onRequest
    }
  }
})
</script>

