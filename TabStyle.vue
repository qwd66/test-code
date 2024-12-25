<template>
  <div>
    <el-form ref="form" :model="form" :rules="formRules" inline>
      <el-form-item label="" prop="associateList">
        <CascadeSelector
          v-model="form.associateList"
          api-key="platformSiteShop"
          not-required-params
          :need-list="[1]"
          placeholder="平台 / 站点 / 店铺"
          @change="
            form.styleIdList=[]
          "
        />
      </el-form-item>
      <el-form-item label="" prop="addDataType">
        <erp-selection
          v-model="form.addDataType"
          :select-options="[
            { label: 'SP', key: 'SP', value: 'SP',},
            { label: 'SD', key: 'SD', value: 'SD',},
            { label: 'SB', key: 'SB', value: 'SB',},
          ]"
          placeholder="广告类型"
          multiple
        />
      </el-form-item>
      <el-form-item label="" prop="combatTeamList">
        <ErpSelection
          v-model="form.combatTeamList"
          placeholder="CT"
          multiple
          :select-options="ctList"
          @change="
            form.styleIdList=[]
          "
        />
      </el-form-item>
      <el-form-item label="" prop="bu">
        <CascadeSelector
          v-model="form.bu"
          :props="{ multiple: true }"
          api-key="BUTree"
          :permission="false"
          collapse-tags
          placeholder="BU"
          @change="
            form.styleIdList=[]
          "
        />
      </el-form-item>
      <el-form-item label="" prop="categoryIdList">
        <CascadeSelector
          v-model="form.categoryIdList"
          collapse-tags
          :permission="false"
          api-key="categoryTreeRefactor"
          :props="{emitPath:false,multiple:true}"
          placeholder="类目"
        />
      </el-form-item>
      <el-form-item label="" prop="styleOpNameList">
        <ErpSelection
          v-model="form.styleOpNameList"
          api-key="operationsUser"
          multiple
          placeholder="销售负责人"
          @change="
            form.styleIdList=[]
          "
        />
      </el-form-item>
      <el-form-item label="" prop="styleIdList">
        <virtualized-select
          v-model="form.styleIdList"
          :options="styleList"
          :props="{
            label: 'style',
            value: 'styleId',
          }"
          collapse-tags
          multiple
          placeholder="Style"
        />
      </el-form-item>
      <el-form-item label="" prop="time">
        <el-date-picker
          v-model="form.time"
          :editable="false"
          align="center"
          value-format="yyyy-MM-dd"
          format="yyyy-MM-dd"
          end-placeholder="结束时间"
          range-separator="~"
          start-placeholder="开始时间"
          type="daterange"
        />
      </el-form-item>
      <div style="margin: 0px 0px 18px 0px; display: inline-block">
        <el-button type="primary" :loading="searchLoading" @click="handleQuery()">查询</el-button>
        <el-button @click="handleReset()">重置</el-button>
      </div>
    </el-form>
    <vxe-toolbar custom>
      <template #buttons />
      <template v-slot:tools>
        <i class="el-icon-download" style="margin-right: 3px;font-size: 20px;" @click="handleExport" />
      </template>
    </vxe-toolbar>
    <vxe-table
      id="advertising-proportion-analysis-new-style"
      ref="table"
      v-loading="searchLoading"
      :max-height="maxHeight"
      :min-height="300"
      :custom-config="{storage: true,checkMethod: checkColumnMethod}"
      class="xTable"
      align="center"
      :data="tableData"
      show-overflow
      :scroll-x="{ enabled: true }"
      :scroll-y="{ enabled: true }"
      :row-config="{ isHover: true,height: 80 }"
      :column-config="{ resizable: true,maxFixedSize:20 }"
      :show-footer="true"
      :footer-data="footerData"
      :sort-config="{ remote: true }"
      @sort-change="handleSort"
    >
      <vxe-table-column field="styleName" title="Style" min-width="180" fixed="left">
        <template #default="{ row }">
          <span style="display: flex;align-items:center">
            <el-image style="width: 70px; height: 70px;margin-right: 5px;" :src="row.styleImageUrl" fit="fit" />
            <span>{{ row.styleName }}</span>
          </span>
        </template>
      </vxe-table-column>
      <vxe-table-column field="combatTeam" title="CT" min-width="100" />
      <vxe-table-column field="season" title="季节" min-width="100" />
      <vxe-table-column field="impressions" title="广告曝光量" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>广告曝光量</span>
          <el-tooltip effect="dark" content="广告被展示的次数" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="clicks" title="广告点击量" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>广告点击量</span>
          <el-tooltip effect="dark" content="广告被点击的次数" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="clickRate" title="广告点击率（CTR）" min-width="170" sortable :formatter="formatter">
        <template #header>
          <span>广告点击率（CTR）</span>
          <el-tooltip effect="dark" content="广告点击量/广告曝光量*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="appSessions" title="Sessions-PC" min-width="150" sortable :formatter="formatter">
        <template #header>
          <span>Sessions-APP</span>
          <el-tooltip effect="dark" content="APP端访问商品详情页的会话次数" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="webSessions" title="Sessions-Browser" min-width="160" sortable :formatter="formatter">
        <template #header>
          <span>Sessions-Browser</span>
          <el-tooltip effect="dark" content="Web端访问商品详情页的会话次数" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="sessions" title="Sessions" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>Sessions</span>
          <el-tooltip effect="dark" content="访问商品详情页的会话次数" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="naturalSessions" title="自然流量" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>自然流量</span>
          <el-tooltip effect="dark" content="Sessions - 广告点击量" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="naturalSessionsRate" title="自然流量占比" min-width="140" sortable :formatter="formatter">
        <template #header>
          <span>自然流量占比</span>
          <el-tooltip effect="dark" content="自然流量/Sessions*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="pv" title="PV" min-width="100" sortable :formatter="formatter">
        <template #header>
          <span>PV</span>
          <el-tooltip effect="dark" content="商品详情页被浏览的次数" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="spend" title="广告花费" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>广告花费</span>
          <el-tooltip effect="dark" content="广告被展示/点击后产生的费用" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="spendRate" title="广告花费占比" min-width="140" sortable :formatter="formatter">
        <template #header>
          <span>广告花费占比</span>
          <el-tooltip effect="dark" content="当前行广告花费/合计广告花费*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="cpc" title="CPC" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>CPC</span>
          <el-tooltip effect="dark" content="广告花费/广告点击量" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="items" title="总订单量" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>总订单量</span>
          <el-tooltip effect="dark" content="商品的总订单量" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="adOrders" title="广告订单量" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>广告订单量</span>
          <el-tooltip effect="dark" content="广告带来的订单，包含广告商品和由广告带来的非广告商品的订单" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="actualOrders" title="实际广告订单量" min-width="140" sortable :formatter="formatter" />
      <vxe-table-column field="naturalOrders" title="自然订单" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>自然订单</span>
          <el-tooltip effect="dark" content="总订单量 - 广告订单量" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="naturalOrdersRate" title="自然订单占比" min-width="140" sortable :formatter="formatter">
        <template #header>
          <span>自然订单占比</span>
          <el-tooltip effect="dark" content="自然订单/总订单量*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="adUnits" title="广告销量" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>广告销量</span>
          <el-tooltip effect="dark" content="广告带来的销量，包含广告商品和由广告带来的非广告商品的销量" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="actualSalesVolume" title="实际广告销量" min-width="120" sortable :formatter="formatter" />
      <vxe-table-column field="orders" title="总销量" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>总销量</span>
          <el-tooltip effect="dark" content="商品的总销量" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="adOrdersRate" title="广告销量占比" min-width="140" sortable :formatter="formatter">
        <template #header>
          <span>广告销量占比</span>
          <el-tooltip effect="dark" content="（当前行广告销量/合计广告销量）*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="adConversion" title="广告转化率（CVR）" min-width="170" sortable :formatter="formatter">
        <template #header>
          <span>广告转化率（CVR）</span>
          <el-tooltip effect="dark" content="广告订单量/广告点击量*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="tConversion" title="总转化率" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>总转化率</span>
          <el-tooltip effect="dark" content="（总订单量/Sessions）*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="adSales" title="广告销售额" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>广告销售额</span>
          <el-tooltip effect="dark" content="广告带来的订单销售额，包含广告商品和由广告带来的非广告商品的订单销售额" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="actualSales" title="实际广告销售额" min-width="150" sortable :formatter="formatter" />
      <vxe-table-column field="adSalesRate" title="广告销售额占比" min-width="150" sortable :formatter="formatter">
        <template #header>
          <span>广告销售额占比</span>
          <el-tooltip effect="dark" content="当前行广告销售额/合计广告销售额*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="sales" title="总销售额" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>总销售额</span>
          <el-tooltip effect="dark" content="商品的总销售额" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="salesRate" title="总销售额占比" min-width="140" sortable :formatter="formatter">
        <template #header>
          <span>总销售额占比</span>
          <el-tooltip effect="dark" content="当前行总销售额/合计总销售额*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="acos" title="ACOS" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>ACOS</span>
          <el-tooltip effect="dark" content="广告花费/广告销售额*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="roas" title="ROAS" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>ROAS</span>
          <el-tooltip effect="dark" content="广告销售额/广告花费" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="tacos" title="广销比（TACOS）" min-width="160" sortable :formatter="formatter">
        <template #header>
          <span>广销比（TACOS）</span>
          <el-tooltip effect="dark" content="广告花费/总销售额*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="aSoAS" title="ASoAS" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>ASoAS</span>
          <el-tooltip effect="dark" content="广告销售额/总销售额*100%" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="cpo" title="CPO" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>CPO</span>
          <el-tooltip effect="dark" content="广告花费/广告订单量" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="avgPrice" title="平均价格" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>平均价格</span>
          <el-tooltip effect="dark" content="总销售额/总销量" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="returnRate" title="退货率" min-width="120" sortable :formatter="formatter" />
      <vxe-table-column field="inventory" title="FBA可售" min-width="120" sortable :formatter="formatter" />
      <vxe-table-column field="turnoverDays" title="周转天数" min-width="120" sortable :formatter="formatter">
        <template #header>
          <span>周转天数</span>
          <el-tooltip effect="dark" content="FBA可售/平均销量，平均销量等于，条件范围内销量/销量大于0的天数" placement="top">
            <i class="el-icon-question" />
          </el-tooltip>
        </template>
      </vxe-table-column>
      <vxe-table-column field="profitRate" title="利润率" min-width="120" sortable :formatter="formatter" />
      <vxe-table-column field="operate" title="操作" width="100" fixed="right">
        <template #default="{ row }">
          <i class="vxe-icon-chart-line " style="font-size: 18px;color: #409eff" @click="handlShowDetail(row)" />
        </template>
      </vxe-table-column>
    </vxe-table>

    <DialogDetail :visible.sync="detailVisible" :detail-row="detailRow" :save-form="saveForm" />
  </div>
</template>
<script>
import ErpSelection from '@/components/ErpSelection'
import CascadeSelector from '@/components/CascadeSelector'
import { debounceGetTableMaxHeight, getCommodityDictByKey } from '@/utils'
import {
  getNewAdDataPage, exportNewAdDataPage,
} from '@/api/erp/amazon-advertise'
import { mapGetters } from 'vuex'
import DialogDetail from './DialogDetail.vue'
import { getStyleList } from '@/api/erp-select'
import VirtualizedSelect from '@/components/VirtualizedSelect'
import {
  precentField, symbolField,
} from './config'
import { cloneDeep } from 'lodash'

import dayjs from 'dayjs'
import 'dayjs/locale/zh-cn'
dayjs.locale('zh-cn')

const dataType = 'style'

export default {
  components: {
    ErpSelection, CascadeSelector, VirtualizedSelect,
    DialogDetail,
  },
  props: {
    styleParams: {
      type: Object,
      default: () => {},
    },
  },
  data() {
    return {
      form: {
        associateList: [1, 1, 1],
        addDataType: [],
        combatTeamList: [],
        bu: [],
        categoryIdList: [],
        styleIdList: [],
        styleOpNameList: [],
        time: [dayjs().subtract(6, 'day').format('YYYY-MM-DD'), dayjs().format('YYYY-MM-DD')],
        orderByFiled: 'spend',
        orderByType: 'desc',
      },
      formRules: {},
      searchLoading: false,
      pager: { current: 1, size: 20, total: 0 },
      pagerConfig: ['current', 'size', 'total'],
      tableData: [],
      maxHeight: 500,
      xTableRef: null,
      queryChange: false,
      saveForm: {},
      styleList: [],
      footerData: [],

      detailVisible: false,
      detailRow: {},
    }
  },
  computed: {
    ...mapGetters('dictionary', ['wmsAllCommodityDict']),
    ctList() {
      return getCommodityDictByKey({
        key: 'COMBAT_TEAM',
        dict: this.wmsAllCommodityDict,
      })
    },
    queryParams() {
      const [platformId, siteId, shopId] = this.form.associateList || []
      const [startDate = '', endDate = ''] = this.form.time || []
      // 如果选了SB就得加入一个SBV,后端需要
      const addDataType = cloneDeep(this.form.addDataType)
      if (addDataType.includes('SB')) {
        addDataType.push('SBV')
      }
      return {
        ...this.form,
        platformId, siteId, shopId,
        startDate,
        endDate,
        buIdList: this.buIdAndGroupId.groupId,
        dataType,
        addDataType,
      }
    },
    buIdAndGroupId() {
      const _buId = new Set()
      const _groupId = new Set()
      if (this.form.bu.length) {
        this.form.bu.forEach(item => {
          item?.[0] && _buId.add(item[0])
          item?.[1] && _groupId.add(item[1])
        })
      }
      return { groupId: [..._groupId], buId: [..._buId] }
    },
    platformSiteShopParams() {
      const [platformId, siteId, shopId] = this.form.associateList || []
      return {
        platformId: platformId ? [platformId] : [],
        siteId: siteId ? [siteId] : [],
        shopId: shopId ? [shopId] : [],
      }
    },
    styleSelectParams() {
      return {
        ...this.platformSiteShopParams,
        buId: this.buIdAndGroupId.buId, groupId: this.buIdAndGroupId.groupId,
        combatTeamList: this.form.combatTeamList,
        selectedUser: this.form.styleOpNameList,
      }
    },
  },
  watch: {
    form: {
      handler() {
        this.queryChange = true
      },
      deep: true,
    },
    styleSelectParams: {
      handler() {
        this.getStyleList()
      },
    },
    styleParams: {
      handler() {
        this.getQueryParams()
        this.handleQuery()
      },
      immediate: true,
    },
  },
  mounted() {
    this.$nextTick(() => {
      this.xTableRef = this.$refs.table

      setTimeout(() => {
        console.log('this.$refs.table', this.$refs.table)
        this.xTableRef.sort(this.form.orderByFiled, this.form.orderByType)
      })
    })

    this.getStyleList()
    this.handleQuery()

    this.debounceGetTableMaxHeight = debounceGetTableMaxHeight.bind(this)
    this.debounceGetTableMaxHeight()
    window.addEventListener('resize', this.debounceGetTableMaxHeight)
  },
  methods: {
    handlShowDetail(row) {
      this.detailRow = row
      this.detailVisible = true
    },
    async handleExport() {
      const { success, message } = await exportNewAdDataPage(this.saveForm).finally(() => {
      })
      if (success) {
        this.$message({
          type: 'success',
          dangerouslyUseHTMLString: true,
          message: `${message || '导出成功'} <a target="_blank" style="background-color: #67C23A;color: #FFFFFF;border-radius: 5px;padding: 5px 10px;display: inline-block" href="/data-download/download">去查看</a>`,
          duration: 5000,
        })
      } else {
        this.$message.error(message || '导出失败')
      }
    },
    async getTableData() {
      this.searchLoading = true
      const { datas: { records = [], pager = {}}} = await getNewAdDataPage(this.queryParams).finally(() => {
        this.searchLoading = false
        this.queryChange = false
      })
      this.tableData = records.slice(0, -1).map(item => {
        return {
          ...item,
          dataType,
        }
      })
      this.footerData = []
      const total = records.slice(-1)[0]
      this.footerData.push({
        ...this.formatterFooter(total),
        dataType,
        styleName: '合计',
      })
      this.pager.total = pager?.total || 0
    },
    formatterFooter(total) {
      const totalObj = {}
      Object.entries(total).forEach(([key, value]) => {
        totalObj[key] = this.formatter({ cellValue: value, column: { property: key }, row: total })
      })
      return totalObj
    },
    handleReset() {
      this.$refs?.form?.resetFields()
      this.xTableRef?.clearSort()
      this.form.orderByType = 'desc'
      this.form.orderByFiled = 'spend'
      this.xTableRef?.sort(this.form.orderByFiled, this.form.orderByType)
    },
    handleQuery() {
      this.$refs?.form?.validate(async(valid) => {
        if (!valid) return
        this.saveForm = { ...this.queryParams }
        if (this.queryChange) {
          this.pager.current = 1
        }
        await this.getTableData()
      })
    },
    getQueryParams() {
      if (Object.values(this.styleParams).length) {
        this.handleReset()
        const { associateList, time, addDataType, combatTeamList, categoryIdList } = this.styleParams
        this.form = {
          ...this.form,
          associateList,
          time,
          addDataType,
          combatTeamList,
          categoryIdList,
        }
      }
    },
    async getStyleList() {
      const { data = [] } = await getStyleList(this.styleSelectParams)
      this.styleList = Object.freeze(data)
    },
    handleSort({ field, order }) {
      this.form.orderByFiled = field
      this.form.orderByType = order
      this.handleQuery()
    },
    checkColumnMethod({ column }) {
      if (column.property === 'combatTeam') return false
      return true
    },
    formatter({ cellValue, column, row }) {
      const cellValueNumber = +cellValue
      const cellValueNumberToLocaleString = cellValueNumber?.toLocaleString()
      if (cellValueNumber && precentField.includes(column.property)) {
        return `${cellValueNumberToLocaleString} %`
      } else if (cellValueNumber && symbolField.includes(column.property)) {
        return `${row?.currency || ''} ${cellValueNumberToLocaleString}`
      } else if (cellValueNumber) {
        return cellValueNumberToLocaleString
      } else {
        return cellValue
      }
    },
  },
}
</script>
