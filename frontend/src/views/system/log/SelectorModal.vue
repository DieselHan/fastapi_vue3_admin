<template>
  <a-modal title="选择创建人" v-model:open="openModal" :width="1200" :destroyOnClose="true" style="top: 30px">
    <template #footer>
      <a-button @click="handleModalCancel">取消</a-button>
      <a-button @click="handleModalClear">清空</a-button>
      <a-button type="primary" @click="handleModalSumbit">确定</a-button>
    </template>

    <div class="table-search-wrapper">
      <a-card :bordered="true">
        <a-form :model="queryState" @finish="onFinish">
          <a-flex wrap="wrap" gap="middle">
              <a-form-item name="creator_name" label="姓名" >
                <a-input v-model:value="queryState.name" placeholder="请输入姓名" allowClear style="width: 200px;"></a-input>
              </a-form-item>
              
              <a-form-item name="available" label="状态" >
                <a-select v-model:value="queryState.available" placeholder="请选择状态" allowClear style="width: 200px;">
                  <a-select-option value="true">启用</a-select-option>
                  <a-select-option value="false">停用</a-select-option>
                </a-select>
              </a-form-item>

              <a-button type="primary" html-type="submit" :loading="tableLoading">查询</a-button>
              <a-button  @click="resetFields">重置</a-button>
          </a-flex>
        </a-form>
      </a-card>
    </div>

    <div class="table-wrapper">
      <a-card title="用户列表" :bordered="true" 
        :bodyStyle="{ padding: '0 24px' }">
        <a-table 
          :rowKey="record => record.id" 
          :columns="columns" 
          :data-source="dataSource" 
          :loading="tableLoading"
          :row-selection="rowSelection" 
          @change="handleTableChange" 
          :pagination="pagination"
          :scroll="{ x: 500, y: 'calc(100vh - 500px)' }"
          :style="{ minHeight: 'calc(100vh - 450px)' }"
          >
          <template v-slot:bodyCell="{ column, record, index }">
            <template v-if="column.dataIndex === 'index'">
              <span>{{ (pagination.current - 1) * pagination.pageSize + index + 1 }}</span>
            </template>
            <template v-if="column.dataIndex === 'available'">
              <span>
                <a-badge :status="record.available ? 'processing': 'error'" :text="record.available ? '启用' : '停用'" />
              </span>
            </template>
          </template>
        </a-table>
      </a-card>
    </div>
  </a-modal>
</template>

<script lang="ts" setup>
import { ref, reactive, computed, unref } from 'vue';
import type { TableColumnsType } from 'ant-design-vue';
import { getUserList } from '@/api/system/user'
import type { searchCreatorDataType, creatorType } from './types'


const openModal = ref<boolean>(false);
const tableLoading = ref(false);

const queryState: searchCreatorDataType = reactive({
  name: "",
  available: undefined,
});


const columns: TableColumnsType = [
  {
    title: '序号',
    dataIndex: 'index',
    align: 'center',
    width: 80
  },
  {
    title: '姓名',
    dataIndex: 'name',
    align: 'center'
  },
  {
    title: '状态',
    dataIndex: 'available',
    align: 'center'
  },
  {
    title: '备注',
    dataIndex: 'description',
    align: 'center',
    ellipsis: true,
    width: 500
  }
]
const dataSource = ref<creatorType[]>([]);

const loadingData = () => {
  tableLoading.value = true;
  dataSource.value = [];

  let params = {};
  if (queryState.name) {
    params['name'] = queryState.name
  }
  if (queryState.available !== null && queryState.available !== undefined) {
        params['available'] = queryState.available;
    }
  params['page_no'] = pagination.current
  params['page_size'] = pagination.pageSize

  getUserList(params).then(response => {
    const result = response.data;
    dataSource.value = result.data.items;
    pagination.total = result.data.total;
    pagination.current = result.data.page_no;
    pagination.pageSize = result.data.page_size;
    tableLoading.value = false;
  }).catch(error => {
    tableLoading.value = false;
  })
}

const onFinish = () => {
  pagination.current = 1;
  loadingData();
}

const resetFields = () => {
  Object.keys(queryState).forEach((key: string) => {
    delete queryState[key];
  });
  queryState.available = true;
  pagination.current = 1;
  loadingData();
}

const pagination = reactive({
  current: 1,
  pageSize: 10,
  defaultPageSize: 10,
  showSizeChanger: true,
  total: dataSource.value.length,
  showTotal: (total, range) => `第 ${range[0]}-${range[1]} 条 / 总共 ${total} 条`
})

const handleTableChange = (values: any) => {
  pagination.current = values.current;
  pagination.pageSize = values.pageSize;
  loadingData();
}

const selectedRowKeys = ref<creatorType['id'][]>([]);
const selectedRowName = ref<creatorType['name']>('');

const onSelectChange = (selectingRowKeys: creatorType['id'][], selectingRows: creatorType[]) => {
  selectedRowKeys.value = selectingRowKeys;
  selectedRowName.value = selectingRows[0].name;
}

const rowSelection = computed(() => {
  return {
    selectedRowKeys: unref(selectedRowKeys),
    onChange: onSelectChange,
    hideDefaultSelections: true,
    type: 'radio'
  }
});

const emit = defineEmits(['event']);

const handleModalCancel = () => {
  openModal.value = false;
  Object.keys(queryState).forEach((key: string) => {
    delete queryState[key];
  });
  queryState.available = true;
  pagination.current = 1;
  pagination.pageSize = pagination.defaultPageSize;
  selectedRowKeys.value = [];
  selectedRowName.value = undefined;
}

const handleModalClear = () => {
  handleModalCancel();
  emit('event', selectedRowKeys.value, selectedRowName.value);
}

const handleModalSumbit = () => {
  emit('event', selectedRowKeys.value, selectedRowName.value);
  handleModalCancel();
}


defineExpose({
  openModal,
  selectedRowKeys,
  selectedRowName,
  loadingData
});

</script>

<style lang="scss" scoped>
.table-search-wrapper {
  margin-block-end: 16px;
}
</style>