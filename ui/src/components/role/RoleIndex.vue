<template>
  <div>
    <role-search @ok="searchHandler"></role-search>
    <fish-table :columns="columns" :data="data" :loading="loading" :pagination="pagination" @change="changeHandler">
      <fish-button @click="addHandler" slot="bottomLeft"><i class="fa fa-plus"></i> </fish-button>
    </fish-table>
    <fish-modal :visible.sync="editVisible" title="Role Information...">
      <role-edit :permissions="permissions" :record="record" @ok="updateHandler"></role-edit>
    </fish-modal>
  </div>
</template>
<script>
  import {mapGetters} from 'vuex'
  import RoleSearch from './RoleSearch.vue'
  import RoleEdit from './RoleEdit.vue'

  export default {
    components: {
      RoleEdit,
      RoleSearch},
    data () {
      return {
        editVisible: false,
        record: null,
        queryPayLoad: {},
        currentUser: window.$currentUser,
        columns: [
          {title: '#', type: 'index', width: '50', align: 'center'},
          {title: 'name', key: 'name'},
          {title: 'Permissions', key: 'permissions', render: (h, {permissions}, column) => h('span', {}, Array.isArray(permissions) ? permissions.join(', ') : '')},
          {title: 'Created at', key: 'createdAt'},
          {
            title: 'Operate',
            key: 'operate',
            render: (h, record, column) => {
              if (this.currentUser.hasPermissions('Role', 'update')) {
                return h('a', {on: {click: this.editHandler.bind(null, record)}}, 'Edit')
              } else {
                return h('span', '')
              }
            }
          }
        ]
      }
    },
    mounted () {
      // load data
      this.loadData()
    },
    computed: {
      ...mapGetters({
        data: 'roleItems',
        pagination: 'rolePagination',
        loading: 'roleLoading',
        permissions: 'rolePermissions'
      })
    },
    methods: {
      loadData () {
        this.$store.dispatch('roleQuery', {})
        this.$store.dispatch('rolePermissions')
      },
      editHandler (record) {
        this.record = record
        this.editVisible = true
      },
      addHandler () {
        this.record = null
        this.editVisible = true
      },
      updateHandler (form) {
        // permissions convert list to map
        if (this.record === null) {
          this.$store.dispatch('roleCreate', form)
        } else {
          form['id'] = this.record.id
          this.$store.dispatch('roleUpdate', form)
        }
        this.editVisible = false
      },
      searchHandler (form) {
        this.queryPayLoad = form
        this.$store.dispatch('roleQuery', form)
      },
      changeHandler (currentPage, filters) {
        this.$store.dispatch('roleQuery', Object.assign({page: currentPage}, this.queryPayLoad))
      }
    }
  }
</script>
