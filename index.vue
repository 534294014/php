<template>
  <div class="app-container">
    <div class="filter-container">
      <!--<el-input v-if="formThead.id" v-model="listQuery.id" style="width: 200px;" class="filter-item" placeholder="id" @keyup.enter.native="handleFilter"/>-->
      <!--<el-input v-if="formThead.parent_id" v-model="listQuery.parent_id" style="width: 200px;" class="filter-item" placeholder="父类id" @keyup.enter.native="handleFilter"/>-->
      <el-input v-if="formThead.name" v-model="listQuery.name" style="width: 200px;" class="filter-item" placeholder="分类名" @keyup.enter.native="handleFilter"/>
      <!--<el-input v-if="formThead.status" v-model="listQuery.status" style="width: 200px;" class="filter-item" placeholder="状态" @keyup.enter.native="handleFilter"/>-->
      <el-select v-if="formThead.status" v-model="listQuery.status" style="width: 200px;" clearable class="filter-item" placeholder="状态" @keyup.enter.native="handleFilter">
        <el-option
          v-for="item in status"
          :key="item.value"
          :label="item.label"
          :value="item.value"/>
      </el-select>
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">搜索</el-button>
      <addPage @getLists="getLists"/>
    </div>

    <div class="filter-container">
      <el-checkbox v-model="formThead.id" label="id">id</el-checkbox>
      <el-checkbox v-model="formThead.parent_id" label="父类id">父类id</el-checkbox>
      <el-checkbox v-model="formThead.name" label="分类名">分类名</el-checkbox>
      <el-checkbox v-model="formThead.status" label="状态">状态</el-checkbox>
    </div>
    <tree-table :data="data" :eval-func="func" :eval-args="args" :expand-all="expandAll" border>
      <!--<el-table v-loading="listLoading" :key="tableKey" :data="list" border fit highlight-current-row style="width: 100%;">-->
      <el-table-column v-if="formThead.id" align="center" label="id">
        <template slot-scope="scope">
          <span>{{ scope.row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column v-if="formThead.parent_id" align="center" label="父类id">
        <template slot-scope="scope">
          <span>{{ scope.row.parent_id }}</span>
        </template>
      </el-table-column>
      <el-table-column v-if="formThead.name" align="center" label="分类名">
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column v-if="formThead.status" align="center" label="状态">
        <template slot-scope="scope">
          <span v-if="scope.row.status == 1"><i class="el-icon-circle-check" style="color: #67C23A;"/></span>
          <span v-else><i class="el-icon-circle-close" style="color: #ebb563;"/></span>
        </template>
      </el-table-column>

      <el-table-column align="center" label="操作" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button v-if="scope.row.status===0" type="success" icon="el-icon-check" circle title="invoke" class="buttoncss" @click="invoke(scope.row.id)"/>
          <el-button v-else-if="scope.row.status===1" type="warning" icon="el-icon-close" circle title="forbidden" class="buttoncss" @click="forbidden(scope.row.id)"/>
          <editPage :row_data="scope.row" @getLists="getLists"/>
          <el-button type="danger" icon="el-icon-delete" circle class="buttoncss" title="删除" @click="del(scope.row.id)"/>
        </template>
      </el-table-column>
    <!--</el-table>-->
    </tree-table>
    <div class="pagination-container">
      <el-pagination :current-page="listQuery.page" :page-sizes="[10,20,30, 50]" :page-size="listQuery.limit" :total="total" background layout="total, sizes, prev, pager, next, jumper" @size-change="handleSizeChange" @current-change="handleCurrentChange"/>
    </div>
  </div>
</template>

<script>
import { getList, del, invoke, forbidden } from '@/api/information/category'
import treeTable from '@/components/TreeTable'
import treeToArray from './customEval'
import store from '@/store'
import waves from '@/directive/waves' // 水波纹指令
import addPage from './addPage'
import editPage from './editPage'
export default {
  name: 'ComplexTable',
  directives: {
    waves
  },
  components: { addPage, editPage, treeTable },
  data() {
    return {
      func: treeToArray,
      expandAll: true,
      data: [],
      args: [null, null, 'left'],
      tableKey: 0,
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        id: undefined,
        parent_id: undefined,
        name: undefined,
        status: undefined,
        page: 1,
        limit: 20
      },
      formThead: {
        id: true,
        parent_id: true,
        name: true,
        status: true
      },
      status: [{
        value: '1',
        label: '可用'
      }, {
        value: '0',
        label: '禁用'
      }]
    }
  },
  created() {
    this.getLists()
  },
  methods: {
    getLists() {
      this.listLoading = true
      getList(store.getters.token, this.listQuery).then(response => {
        this.list = response.data.list.data
        this.total = response.data.list.total
        this.data = response.data.list.data
        // console.log(this.data)
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getLists()
    },
    handleSizeChange(val) {
      this.listQuery.limit = val
      this.getLists()
    },
    handleCurrentChange(val) {
      this.listQuery.page = val
      this.getLists()
    },
    invoke(id) {
      invoke(store.getters.token, { id: id }).then(res => {
        this.$message({
          message: '启用',
          type: 'success'
        })
        this.getLists()
      })
    },
    forbidden(id) {
      forbidden(store.getters.token, { id: id }).then(res => {
        this.$message({
          message: '禁用',
          type: 'warning'
        })
        this.getLists()
      })
    },
    del(id) {
      this.$confirm('操作将会删除该记录,是否继续', '提示', {
        confirmButtonText: '是',
        cancelButtonText: '否',
        type: 'warning'
      }).then(() => {
        del(store.getters.token, { id: id }).then(res => {
          this.$message({
            type: 'success',
            message: '成功!'
          })
          this.getLists()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '失败!'
        })
      })
    }
  }
}
</script>
