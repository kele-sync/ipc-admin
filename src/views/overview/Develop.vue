
<template>
  <page-header-wrapper>
    <a-card :bordered="false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="48">
            <a-col :md="8" :sm="24">
              <a-form-item label="省份">
                <a-input v-model="queryParam.id" placeholder="" />
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="城市">
                <a-input v-model="queryParam.id" placeholder="" />
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="行业">
                <a-select v-model="queryParam.status" placeholder="请选择" default-value="0">
                  <a-select-option value="0">全部</a-select-option>
                  <a-select-option value="1">关闭</a-select-option>
                  <a-select-option value="2">运行中</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="24">
              <span
                class="table-page-search-submitButtons"
                :style="(advanced && { float: 'right', overflow: 'hidden' }) || {}"
              >
                <a-button type="primary" @click="$refs.table.refresh(true)">查询</a-button>
                <a-button style="margin-left: 8px" @click="() => (this.queryParam = {})">导入</a-button>
                <!-- <a-button style="margin-left: 8px" @click="() => this.queryParam = {}">导出</a-button> -->
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <div class="table-operator">
        <a-dropdown v-if="selectedRowKeys.length > 0">
          <a-menu slot="overlay">
            <a-menu-item key="1"><a-icon type="delete" />添加至待开发客户</a-menu-item>
            <!-- lock | unlock -->
            <a-menu-item key="2"><a-icon type="lock" />锁定</a-menu-item>
          </a-menu>
          <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /> </a-button>
        </a-dropdown>
      </div>
      <s-table
        ref="table"
        size="default"
        rowKey="key"
        :columns="columns"
        :data="loadData"
        :alert="false"
        showPagination="auto"
      >
        <span slot="serial" slot-scope="text, record, index">
          {{ index + 1 }}
        </span>
        <span slot="status" slot-scope="text">
          <a-badge :status="text | statusTypeFilter" :text="text | statusFilter" />
        </span>
        <span slot="description" slot-scope="text">
          <ellipsis :length="4" tooltip>{{ text }}</ellipsis>
        </span>

        <span slot="action" slot-scope="text, record">
          <template>
            <a @click="handleEdit(record)">查看详情</a>
            <a-divider type="vertical" />
            <a-dropdown>
              <a class="ant-dropdown-link" @click="e => e.preventDefault()"> 更多 <a-icon type="down" /> </a>
              <a-menu slot="overlay">
                <a-menu-item>
                  <a href="javascript:;" @click="addToDevelop()">转至意向客户</a>
                </a-menu-item>
                <a-menu-item>
                  <a href="javascript:;">更新沟通时间</a>
                </a-menu-item>
                <a-menu-item>
                  <a href="javascript:;">转至客户</a>
                </a-menu-item>
              </a-menu>
            </a-dropdown>
          </template>
        </span>
      </s-table>
      <a-drawer title="企业详情" placement="right" width="640" @close="visible = false" :visible="visible">
        <a-descriptions title="常州天正工业" :column="1">
          <a-descriptions-item label="域名">
            <a href="https:www.baidu.com">https:www.baidu.com</a>
          </a-descriptions-item>
          <a-descriptions-item label="备案人">
            扶工
          </a-descriptions-item>
          <a-descriptions-item label="网站名称">
            天正工业官网
          </a-descriptions-item>
          <a-descriptions-item label="更新时间">
            2020-1-10
          </a-descriptions-item>
          <a-descriptions-item label="注册地址">
            No. 18, Wantang Road, Xihu District, Hangzhou, Zhejiang, China
          </a-descriptions-item>
          <a-descriptions-item label="上次联系时间">
            2020-1-10
          </a-descriptions-item>
          <a-descriptions-item label="电话二维码">
            <vue-qrcode value="Hello, World!" :options="{ width: 200 }"></vue-qrcode>
          </a-descriptions-item>

        </a-descriptions>
        <div>
          <a-button type="primary">
            复制信息
          </a-button>
          <a-dropdown>
            <a-menu slot="overlay">
              <a-menu-item key="1"> <a-icon type="user" />意向客户</a-menu-item>
              <a-menu-item key="2"> <a-icon type="user" />删除</a-menu-item>
            </a-menu>

            <a-button style="margin-left: 8px">转至 <a-icon type="down" /> </a-button>
          </a-dropdown>
        </div>
        <a-divider></a-divider>
        <div>
          <a-form-model labelAlign="left" :model="form" :label-col="labelCol" :wrapper-col="wrapperCol">
            <a-form-model-item label="备注">
              <a-input v-model="form.desc" type="textarea" :autosize="{ minRows: 6, maxRows: 8 }" />
            </a-form-model-item>
            <a-form-model-item >
              <a-button type="primary" @click="onSubmit">
                保存
              </a-button>
            </a-form-model-item>
          </a-form-model>
        </div>
      </a-drawer>
    </a-card>
  </page-header-wrapper>
</template>

<script>
import moment from 'moment'
import { STable, Ellipsis } from '@/components'
import { getRoleList, getServiceList } from '@/api/manage'
import VueQrcode from '@chenfengyuan/vue-qrcode'

const columns = [
  {
    title: '#',
    scopedSlots: { customRender: 'serial' }
  },
  {
    title: '单位名称',
    dataIndex: 'description',
    scopedSlots: { customRender: 'description' }
  },
  {
    title: '备案人',
    dataIndex: 'callNo',
    sorter: true,
    customRender: text => text + ' 次'
  },
  {
    title: '上次沟通时间',
    dataIndex: 'updatedAt',
    sorter: true
  },
  {
    title: '联系方式',
    dataIndex: 'updatedAt',
    sorter: true
  },
  {
    title: '操作',
    dataIndex: 'action',
    scopedSlots: { customRender: 'action' }
  }
]

const statusMap = {
  0: {
    status: 'default',
    text: '关闭'
  },
  1: {
    status: 'processing',
    text: '运行中'
  },
  2: {
    status: 'success',
    text: '已上线'
  },
  3: {
    status: 'error',
    text: '异常'
  }
}

export default {
  name: 'TableList',
  components: {
    STable,
    Ellipsis,
    VueQrcode
  },
  data () {
    this.columns = columns
    return {
      // create model
      visible: false,
      confirmLoading: false,
      mdl: null,
      labelCol: { span: 2 },
      wrapperCol: { span: 18 },
      form: {
        name: '',
        region: undefined,
        date1: undefined,
        delivery: false,
        type: [],
        resource: '',
        desc: ''
      },
      // 高级搜索 展开/关闭
      advanced: true,
      // 查询参数
      queryParam: {},
      // 加载数据方法 必须为 Promise 对象
      loadData: parameter => {
        const requestParameters = Object.assign({}, parameter, this.queryParam)
        console.log('loadData request parameters:', requestParameters)
        return getServiceList(requestParameters).then(res => {
          return res.result
        })
      },
      selectedRowKeys: [],
      selectedRows: []
    }
  },
  filters: {
    statusFilter (type) {
      return statusMap[type].text
    },
    statusTypeFilter (type) {
      return statusMap[type].status
    }
  },
  created () {
    getRoleList({ t: new Date() })
  },
  computed: {
    rowSelection () {
      return {
        selectedRowKeys: this.selectedRowKeys,
        onChange: this.onSelectChange
      }
    }
  },
  methods: {
       onSubmit () {
      console.log('submit!', this.form)
    },
    addToDevelop () {
      this.$message.info('已添加至待开发客户')
    },
    handleAdd () {
      this.mdl = null
      this.visible = true
    },
    handleEdit (record) {
      this.visible = true
      this.mdl = { ...record }
    },
    handleOk () {
      const form = this.$refs.createModal.form
      this.confirmLoading = true
      form.validateFields((errors, values) => {
        if (!errors) {
          console.log('values', values)
          if (values.id > 0) {
            // 修改 e.g.
            new Promise((resolve, reject) => {
              setTimeout(() => {
                resolve()
              }, 1000)
            }).then(res => {
              this.visible = false
              this.confirmLoading = false
              // 重置表单数据
              form.resetFields()
              // 刷新表格
              this.$refs.table.refresh()

              this.$message.info('修改成功')
            })
          } else {
            // 新增
            new Promise((resolve, reject) => {
              setTimeout(() => {
                resolve()
              }, 1000)
            }).then(res => {
              this.visible = false
              this.confirmLoading = false
              // 重置表单数据
              form.resetFields()
              // 刷新表格
              this.$refs.table.refresh()

              this.$message.info('新增成功')
            })
          }
        } else {
          this.confirmLoading = false
        }
      })
    },
    handleCancel () {
      this.visible = false

      const form = this.$refs.createModal.form
      form.resetFields() // 清理表单数据（可不做）
    },
    handleSub (record) {
      if (record.status !== 0) {
        this.$message.info(`${record.no} 订阅成功`)
      } else {
        this.$message.error(`${record.no} 订阅失败，规则已关闭`)
      }
    },
    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
    toggleAdvanced () {
      this.advanced = !this.advanced
    },
    resetSearchForm () {
      this.queryParam = {
        date: moment(new Date())
      }
    }
  }
}
</script>
