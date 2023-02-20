<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div class="q-pa-md">
    <q-table
      :data="data"
      :columns="columns"
      row-key="key"
      :pagination.sync="pagination"
      :loading="loading"
      :filter="filter"
      @request="onRequest"
      binary-state-sort
    >
      <template v-slot:top-right>
        <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </template>
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td>
            <q-btn size="sm" color="accent" round dense @click="props.expand = !props.expand" :icon="props.expand ? 'remove' : 'add'" />
          </q-td>
          <q-td key="des" :props="props">
            {{ props.row.des }}
          </q-td>
          <q-td key="key" :props="props">
            {{ props.row.key }}
            <!--<q-popup-edit v-model="props.row.name" buttons>-->
            <!--<q-input v-model="props.row.name" dense autofocus counter />-->
            <!--</q-popup-edit>-->
          </q-td>
          <q-td key="value" :props="props">
            {{ props.row.value }}
            <q-popup-edit v-model="props.row.value" :validate="val => val.length > 0">
              <template v-slot="{ initialValue, value, emitValue, validate, set , cancel }">
                <q-input
                  type="textarea"
                  v-model="props.row.value"
                  autofocus
                  dense
                  hint="填写value"
                  :rules="[
              val => validate(value) || 'More than 5 chars required'
            ]"
                  @input="emitValue"
                >
                  <template v-slot:after>
                    <q-btn flat dense color="negative" icon="cancel" @click.stop="cancel" label="取消"/>
                    <q-btn flat dense color="positive" icon="check_circle" @click="setInput(props.row)" @click.stop="set" label="保存"
                           :disable="validate(value) === false ||
                           initialValue === value" />
                  </template>
                </q-input>
              </template>
            </q-popup-edit>
          </q-td>
          <q-td key="valueDes" :props="props">
            <div class="text-pre-wrap">{{ props.row.valueDes }}</div>
          </q-td>
        </q-tr>
        <q-tr v-show="props.expand" :props="props">
          <q-td colspan="100%">
            <div class="text-left">This is expand slot for row above: {{ props.row.des }}.</div>
          </q-td>
        </q-tr>
      </template>

    </q-table>
  </div>
</template>
import { Notify } from 'quasar'
<script>
  const baseUrl = 'http://localhost:8082'
  export default {
    props: {
      group: String,
      app: String
    },
    watch: {
      app () {
        if (this.group === '') {
          this.data = []
        } else {
          this.loadData()
        }
      },
      group () {
        if (this.group === '') {
          this.data = []
        } else {
          this.loadData()
        }
      }
    },
    data () {
      return {
        filter: '',
        loading: false,
        pagination: {
          sortBy: 'desc',
          descending: false,
          page: 1,
          rowsPerPage: 5,
          rowsNumber: 5
        },
        columns: [
          { name: 'expand', label: '' },
          {
            name: 'des',
            required: true,
            label: '字段名称',
            align: 'left'
          },
          { name: 'key', align: 'center', label: 'key', field: 'key', sortable: true },
          { name: 'value', label: 'value', field: 'value', sortable: false },
          { name: 'valueDes', label: 'valueDes', field: 'valueDes', sortable: false }
        ],
        data: [],
        original: []
      }
    },
    mounted () {
      // get initial data from server (1st page)
      // this.onRequest({
      //   pagination: this.pagination,
      //   filter: undefined
      // })
    },
    methods: {
      setInput (value) {
        this.$axios.post(baseUrl + '/api/setData', value).then((response) => {
          console.log(response.data)
        })
          .catch(() => {})

        this.$q.notify({
          message: '修改成功',
          icon: 'check_circle',
          color: 'purple',
          position: 'top'
        })
        console.log(value)
      },
      loadData () {
        this.$axios.get(baseUrl + '/api/data', {
          params: {
            app: this.app,
            group: this.group
          }
        }).then((response) => {
          console.log(response.data)
          this.original = response.data
          this.onRequest()
          // this.data.splice(0, this.data.length, ...this.original)
        })
          .catch(() => {})
      },
      onRequest (props) {
        const { page, rowsPerPage, sortBy, descending } = this.pagination
        var filter = ''
        if (props !== undefined && props !== null) {
          filter = props.filter
        }
        this.loading = true
        // emulate server
        // setTimeout(() => {
        // update rowsCount with appropriate value
        this.pagination.rowsNumber = this.getRowsNumberCount(filter)

        // get all rows if "All" (0) is selected
        const fetchCount = rowsPerPage === 0 ? this.pagination.rowsNumber : rowsPerPage

        // calculate starting row of data
        const startRow = (page - 1) * rowsPerPage

        // fetch data from "server"
        const returnedData = this.fetchFromServer(startRow, fetchCount, filter, sortBy, descending)

        // clear out existing data and add new
        this.data.splice(0, this.data.length, ...returnedData)

        // don't forget to update local pagination object
        this.pagination.page = page
        this.pagination.rowsPerPage = rowsPerPage
        this.pagination.sortBy = sortBy
        this.pagination.descending = descending

        // ...and turn of loading indicator
        this.loading = false
        // }, 1500)
      },
      // emulate ajax call
      // SELECT * FROM ... WHERE...LIMIT...
      fetchFromServer (startRow, count, filter, sortBy, descending) {
        const data = filter
          ? this.original.filter(row => row.key.includes(filter))
          : this.original.slice()

        // handle sortBy
        if (sortBy) {
          this.descending = !this.descending
          const sortFn = sortBy === 'desc'
            ? (this.descending
                ? (a, b) => (a.key > b.key ? -1 : a.key < b.key ? 1 : 0)
                : (a, b) => (a.key > b.key ? 1 : a.key < b.key ? -1 : 0)
            )
            : (this.descending
                ? (a, b) => (parseFloat(b[sortBy]) - parseFloat(a[sortBy]))
                : (a, b) => (parseFloat(a[sortBy]) - parseFloat(b[sortBy]))
            )
          data.sort(sortFn)
        }

        return data.slice(startRow, startRow + count)
      },

      // emulate 'SELECT count(*) FROM ...WHERE...'
      getRowsNumberCount (filter) {
        if (!filter) {
          return this.original.length
        }
        let count = 0
        this.original.forEach((treat) => {
          if (treat.key.includes(filter)) {
            ++count
          }
        })
        return count
      }
    }
  }
</script>
