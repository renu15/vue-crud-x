<template>
  <div id="not-needed">
    <v-layout row wrap class="px-2">
      <v-flex xs12>
        <vue-crud-x
          ref="book-pages-table"
          :parentId="parentId"
          v-bind="pageDefs"
        >
          <template v-slot:filter="{ filters, parentId }">
            <div>{{ filters }}</div>
            <hr/>
            <div>{{ !!parentId }}</div>
          </template>
        </vue-crud-x>
      </v-flex>
    </v-layout>
  </div>
</template>

<script>
import { http } from '@/axios'

export default {
  name: 'book-pages',
  data () {
    return {
      parentId: null,
      pageDefs: {
        title: 'Book-pages',
        vtable: {
          headers: [
            // { text: 'Action', value: '', fixed: true, sortable: false, class: 'pa-1' },
            {
              text: 'ID',
              value: 'id',
              align: 'left',
              sortable: false,
              action: true
            },
            {
              text: 'Page Content',
              value: 'content',
              align: 'left',
              sortable: false,
              edit: { type: 'v-text-field' }
            }
          ]
        },
        onRowClick: () => { console.log('override row click') },
        filters: null,
        inline: {
          update: true,
          create: true,
          delete: true
        },
        // form: null,
        form: {
          'id': { value: '', default: '', hidden: 'all' },
          'content': { value: '', default: '', type: 'v-text-field' },
          'bookId': { value: '', default: '', hidden: 'all' }
        },
        crud: {
          // CRUD
          find: async ({ parentId, pagination, filters = {}, sorters = {} }) => {
            let records = []
            let totalRecords = 0
            const { page, itemsPerPage } = pagination // sortBy, descending
            let params = { page: page > 0 ? page - 1 : 0, limit: itemsPerPage, ...filters, sort: sorters } // set query params
            try {
              const { data: { results, total } } = await http.get(`/api/books/${parentId}/pages`, { params })
              records = results
              totalRecords = total
              return { status: 200, data: { records, totalRecords } }
            } catch (e) {
              return { status: e.response.status, error: e.toString() }
            }
          },
          findOne: async (id) => {
            try {
              const { data } = await http.get(`/api/pages/${id}`)
              return { status: 200, data }
            } catch (e) {
              return { status: e.response.status, error: e.toString() }
            }
          },
          create: async (payload) => {
            try {
              let { record, parentId } = payload
              // let { record: { content }, parentId } = payload
              const { data } = await http.post(`/api/books/${parentId}/pages`, record)
              return { status: 201, data }
            } catch (e) {
              return { status: e.response.status, error: e.toString() }
            }
          },
          update: async (payload) => {
            try {
              let { record: { id, content } } = payload
              const { data } = await http.patch(`/api/pages/${id}`, { content })
              return { status: 200, data }
            } catch (e) {
              return { status: e.response.status, error: e.toString() }
            }
          },
          'delete': async (id) => {
            try {
              const { data } = await http.delete(`/api/pages/${id}`)
              return { status: 200, data }
            } catch (e) {
              return { status: e.response.status, error: e.toString() }
            }
          } // done
        }
      }
    }
  },
  created () {
    // this.pageDefs.form = null // uncomment to test without forms
    this.pageDefs.inline.add = false // comment to test without forms
    this.parentId = this.$route.params.id
  }
}

</script>
