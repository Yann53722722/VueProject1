<template>
  <el-card>
    <el-row :gutter="20" class="header">
      <el-col :span="7">
        <el-input
          :placeholder="$t('table.placeholder')"
          clearable
          v-model="queryFrom.query"
        ></el-input>
      </el-col>
      <el-button type="primary" :icon="Search" @click="initGetUsersList">{{
        $t('table.search')
      }}</el-button>
      <el-button type="primary" @click="handleDialogValue">{{
        $t('table.adduser')
      }}</el-button>
    </el-row>

    <el-table :data="tableData" stripe style="width: 100%">
      <el-table-column
        :width="item.width"
        v-for="(item, index) in opstions"
        :key="index"
        :prop="item.prop"
        :label="$t(`table.${item.label}`)"
      >
        <template v-slot="{ row }" v-if="item.prop === 'mg_state'">
          <el-switch v-model="row.mg_state" @change="changeState(row)" />
        </template>
        <template v-slot="{ row }" v-else-if="item.prop === 'create_time'">
          {{ $filters.filterTimes(row.create_time) }}
        </template>
        <template #default v-else-if="item.prop === 'action'">
          <el-button type="primary" size="small" :icon="Edit"></el-button>
          <el-button type="warning" size="small" :icon="Setting"></el-button>
          <el-button type="danger" size="small" :icon="Delete"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      v-model:currentPage="queryFrom.pagenum"
      v-model:page-size="queryFrom.pagesize"
      :page-sizes="[2, 5, 10, 15]"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </el-card>
  <Dialog
    v-model="dialogVisible"
    :dialogTitle="dialogTitle"
    v-if="dialogVisible"
  />
</template>

<script setup>
import { Search, Edit, Setting, Delete } from '@element-plus/icons-vue'
import { ref } from 'vue'
import { getUser, changeUserState } from '@/api/user'
import { opstions } from './options'
import { ElMessage } from 'element-plus'
import { useI18n } from 'vue-i18n'
import Dialog from './components/dialog.vue'

const i18n = useI18n()
const queryFrom = ref({
  query: '',
  pagenum: 1,
  pagesize: 2
})
const tableData = ref([])
const total = ref(0)
const dialogVisible = ref(false)
const dialogTitle = ref('')

const initGetUsersList = async () => {
  const res = await getUser(queryFrom.value)
  tableData.value = res.users
  total.value = res.total
}
initGetUsersList()

const handleSizeChange = (pageSize) => {
  queryFrom.value.pagenum = 1
  queryFrom.value.pagesize = pageSize
  initGetUsersList()
}
const handleCurrentChange = (pageNum) => {
  queryFrom.value.pagenum = pageNum
  initGetUsersList()
}

const handleDialogValue = () => {
  dialogTitle.value = '添加用户'
  dialogVisible.value = true
}

const changeState = async (info) => {
  await changeUserState(info.id, info.mg_state)
  ElMessage({
    message: i18n.t('message.updeteSuccess'),
    type: 'success'
  })
}
</script>

<style lang="scss" scoped>
.header {
  padding-bottom: 16px;
  box-sizing: border-box;
}

::v-deep .el-input__suffix {
  align-items: center;
}
</style>
