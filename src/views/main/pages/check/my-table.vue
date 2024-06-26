<template>
  <div class="layout-container">
    <div class="layout-container-form flex space-between">
      <div class="layout-container-form-handle">
        <el-button type="primary" :icon="Plus" @click="handleAdd">{{ $t('message.common.add') }}</el-button>
        <ExcelUpload @upload-success="handleUploadSuccess" :soversion="soversion" :api="Api" />
      </div>
    </div>
    <div class="layout-container-table">
      <Table ref="table" v-model:page="page" v-loading="loading" :showIndex="true" :showSelection="false"
        :data="tableData" @getTableData="getTableData" @selection-change="handleSelectionChange">
        <el-table-column prop="label" label="硬件名称" align="center" />
        <el-table-column prop="value" label="硬件版本号" align="center" />
        <el-table-column prop="type" label="是否必选" align="center" />
        <el-table-column :label="$t('message.common.handle')" align="center" fixed="right" width="200">
          <template #default="scope">
            <el-button @click="handleEdit(scope.row)">{{ $t('message.common.update') }}</el-button>
            <el-popconfirm :title="$t('message.common.delTip')" @confirm="handleDel([scope.row])">
              <template #reference>
                <el-button type="danger">{{ $t('message.common.del') }}</el-button>
              </template>
            </el-popconfirm>
          </template>
        </el-table-column>
      </Table>
      <Layer :layer="layer" @getTableData="getTableData" v-if="layer.show" />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, inject, watch } from 'vue'
import Table from '@/components/table/index.vue'
import { Page } from '@/components/table/type'
import { getData, del } from '@/api/check'
import Layer from './layer.vue'
import ExcelUpload from '@/components/excelUoload/index.vue'
import { ElMessage } from 'element-plus'
import type { LayerInterface } from '@/components/layer/index.vue'
import { Plus, Search, Delete } from '@element-plus/icons'
export default defineComponent({
  components: {
    Table,
    Layer,
    ExcelUpload
  },
  setup() {
    // 弹窗控制器
    const layer: LayerInterface = reactive({
      show: false,
      title: '新增',
      showButton: true
    })
    // 分页参数, 供table使用
    const page: Page = reactive({
      index: 1,
      size: 20,
      total: 0
    })
    const activeCategory: any = inject('active')
    const loading = ref(true)
    const tableData = ref([])
    const chooseData = ref([])
    const soversion =ref(String)
    const Api = '/api/uploadtwo'
    const handleSelectionChange = (val: []) => {
      chooseData.value = val
    }
    // 获取表格数据
    // params <init> Boolean ，默认为false，用于判断是否需要初始化分页
    const getTableData = (init: boolean) => {
      soversion.value= activeCategory.value.id
      loading.value = true
      if (init) {
        page.index = 1
      }
      let params = {
        soversion_id: activeCategory.value.id,
        page: page.index,
        pageSize: page.size,
      }
      getData(params)
        .then(res => {
          let data = res.data.list
          tableData.value = res.data.list
          page.total = Number(res.data.pager.total)
        })
        .catch(error => {
          tableData.value = []
          page.index = 1
          page.total = 0
        })
        .finally(() => {
          loading.value = false
        })
    }
    // 删除功能
    const handleDel = (data: any) => {
      del(data)
        .then(res => {
          ElMessage({
            type: 'success',
            message: '删除成功'
          })
          getTableData(tableData.value.length === 1 ? true : false)
        })
    }
    const handleUploadSuccess = (response: any) => {
      getTableData(true)
    }
    // 新增弹窗功能
    const handleAdd = () => {
      console.log(soversion);
      layer.config_id = activeCategory.value.id
      layer.title = '新增数据'
      layer.show = true
      delete layer.row
    }
    // 编辑弹窗功能
    const handleEdit = (row: object) => {
      layer.title = '编辑数据'
      layer.row = row
      layer.show = true
    }
    watch(activeCategory, (newVal) => {
      getTableData(true)
    })
    // getTableData(true)
    return {
      Plus,
      Search,
      Delete,
      tableData,
      chooseData,
      loading,
      page,
      layer,
      Api,
      soversion,
      ExcelUpload,
      handleSelectionChange,
      handleAdd,
      handleEdit,
      handleDel,
      getTableData,

      handleUploadSuccess
    }
  }
})
</script>

<style lang="scss" scoped>
.layout-container {
  width: calc(100% - 10px);
  height: 100%;
  margin: 0 0 0 10px;
}
</style>