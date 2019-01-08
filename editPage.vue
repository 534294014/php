<template>
  <div style="display: inline-block;">
    <el-button type="primary" icon="el-icon-edit" class="buttoncss" circle title="修改" @click="getList()"/>
    <el-dialog v-el-drag-dialog :visible.sync="dialogTableVisible" style="text-align: left;" title="修改" @close="close">
      <el-form ref="row_data" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item v-show="show" :label="'id'" prop="id">
          <el-input v-model="row_data.id" placeholder="id"/>
        </el-form-item>
        <el-form-item :label="'父类id'" prop="parent_id">
          <el-select v-model="row_data.parent_id" clearable placeholder="请选择">
            <el-option
              v-for="item in parent_id"
              :key="item.id"
              :label="item.name"
              :value="item.id"/>
          </el-select>
        </el-form-item>
        <el-form-item :label="'分类名'" prop="name">
          <el-input v-model="row_data.name" placeholder="分类名"/>
        </el-form-item>
        <el-form-item :label="'状态'" prop="status">
          <!-- <el-radio-group v-model="row_data.status" placeholder="状态">
            <el-radio :label="0">禁用</el-radio>
            <el-radio :label="1">可用</el-radio>
          </el-radio-group>-->
          <el-radio v-for="sta in status" v-model="row_data.status" :key="sta.id" :label="sta.id" :value="sta.id">{{ sta.name }}</el-radio>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm()">编辑</el-button>
          <el-button @click="close()">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import elDragDialog from '@/directive/el-dragDialog' // base on element-ui
import store from '@/store'
import { edit } from '@/api/information/category'
import { getTopCategory } from '@/api/information/information'
export default {
  name: 'DragDialogDemo',
  directives: { elDragDialog },
  props: {
    row_data: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function() {
        return null
      }
    }
  },
  data() {
    return {
      show: false,
      parent_id: [],
      status: [
        { id: 0, name: '不可用' },
        { id: 1, name: '可用' }
      ],
      dialogTableVisible: false
    }
  },
  created: function() {
    // console.log(this.row_data)
  },
  methods: {
    submitForm() {
      // console.log(this.row_data)
      const data = {}
      data['id'] = this.row_data.id
      data['parent_id'] = this.row_data.parent_id
      data['name'] = this.row_data.name
      data['status'] = this.row_data.status
      edit(store.getters.token, data).then(res => {
        this.close()
      })
    },
    close() {
      this.$emit('getLists')
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
