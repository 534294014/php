<template>
  <div>
    <el-button type="primary" @click="getList()">添加</el-button>
    <el-dialog v-el-drag-dialog :visible.sync="dialogTableVisible" title="添加">
      <el-form ref="row_data" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <!--<el-form-item label="id'" prop="id">
          <el-input v-model="row_data.id" placeholder="id"/>
        </el-form-item>-->
        <el-form-item label="父类" prop="parent_id">
          <!--<el-input v-model="row_data.parent_id" placeholder="父类id"/>-->
          <el-select v-model="row_data.parent_id" clearable placeholder="请选择">
            <el-option
              v-for="item in parent_id"
              :key="item.id"
              :label="item.name"
              :value="item.id"/>
          </el-select>
        </el-form-item>
        <el-form-item label="分类名" prop="name">
          <el-input v-model="row_data.name" placeholder="分类名"/>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="row_data.status" placeholder="状态">
            <el-radio :label="0">禁用</el-radio>
            <el-radio :label="1">可用</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm()">立即创建</el-button>
          <el-button @click="close()">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import elDragDialog from '@/directive/el-dragDialog' // base on element-ui
import { add } from '@/api/information/category'
import { getTopCategory } from '@/api/information/information'
import store from '@/store'
export default {
  name: 'DragDialogDemo',
  directives: { elDragDialog },
  data() {
    return {
      status: '0',
      dialogTableVisible: false,
      parent_id: [],
      row_data: {
      }
    }
  },
  methods: {
    submitForm() {
      add(store.getters.token, this.row_data).then(res => {
        this.$emit('getLists')
        this.close()
      })
    },
    close() {
      this.dialogTableVisible = false
    },
    getList() {
      this.dialogTableVisible = true
      this.listLoading = true
      getTopCategory(store.getters.token, this.listQuery).then(response => {
        // console.log(response.data.data)
        this.parent_id = response.data.data
        const data = []
        data['id'] = 0
        data['name'] = '顶级分类'
        this.parent_id.push(data)
        this.listLoading = false
      })
    }
  }
}
</script>
