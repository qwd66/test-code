<template>
  <div>
    <el-dialog
      title="趋势分析"
      :close-on-click-modal="true"
      :close-on-press-escape="true"
      :visible.sync="visible"
      :before-close="cancel"
      append-to-body
      width="90%"
      top="3vh"
    >
      <div style="display: flex;justify-content: space-between;align-items: center">
        <el-form ref="form" :model="form" :rules="formRules" inline>
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
              @change="getData"
            />
          </el-form-item>
          <el-form-item label="" prop="dateType">
            <el-radio-group v-model="form.dateType" @change="getData">
              <el-radio-button v-for="item in dateTypeMap" :key="item.value" :label="item.label" />
            </el-radio-group>
          </el-form-item>
        </el-form>
        <div>
          <el-button :loading="exportLoading" @click="handleExport">导出</el-button>
        </div>
      </div>
      <div :style="{'max-height': dialogHeight+'px'}" style="overflow-x: hidden;overflow-y: auto;">
        <div ref="chart" style="height: 50vh;width: 100%;margin-top: 10px;" />
        <vxe-table
          ref="table-dialog-ref"
          v-loading="searchLoading"
          max-height="1000"
          class="xTable"
          align="center"
          :data="tableData"
          show-overflow
          :scroll-x="{ enabled: true }"
          :scroll-y="{ enabled: true }"
          :row-config="{ isHover: true }"
          :column-config="{ resizable: true }"
          :show-footer="true"
          :footer-data="footerData"
        >
          <vxe-table-column v-if="detailRow.dataType==='CT'" field="combatTeam" title="CT" min-width="100" fixed="left" />
          <vxe-table-column v-if="detailRow.dataType==='category'" field="categoryName" title="类目" min-width="120" fixed="left" />
          <vxe-table-column v-if="detailRow.dataType==='style'" field="styleName" title="Style" min-width="120" fixed="left" />
          <vxe-table-column field="dateListString" title="日期" min-width="150" sortable fixed="left" />
          <vxe-table-column field="impressions" title="广告曝光量" min-width="120" :formatter="formatter">
            <template #header>
              <span>广告曝光量</span>
              <el-tooltip effect="dark" content="广告被展示的次数" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="clicks" title="广告点击量" min-width="120" :formatter="formatter">
            <template #header>
              <span>广告点击量</span>
              <el-tooltip effect="dark" content="广告被点击的次数" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="clickRate" title="广告点击率（CTR）" min-width="170" :formatter="formatter">
            <template #header>
              <span>广告点击率（CTR）</span>
              <el-tooltip effect="dark" content="广告点击量/广告曝光量*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="appSessions" title="Sessions-PC" min-width="150" :formatter="formatter">
            <template #header>
              <span>Sessions-APP</span>
              <el-tooltip effect="dark" content="APP端访问商品详情页的会话次数" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="webSessions" title="Sessions-Browser" min-width="160" :formatter="formatter">
            <template #header>
              <span>Sessions-Browser</span>
              <el-tooltip effect="dark" content="Web端访问商品详情页的会话次数" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="sessions" title="Sessions" min-width="120" :formatter="formatter">
            <template #header>
              <span>Sessions</span>
              <el-tooltip effect="dark" content="访问商品详情页的会话次数" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="organicTraffic" title="自然流量" min-width="120" :formatter="formatter">
            <template #header>
              <span>自然流量</span>
              <el-tooltip effect="dark" content="Sessions - 广告点击量" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="organicTrafficShare" title="自然流量占比" min-width="140" :formatter="formatter">
            <template #header>
              <span>自然流量占比</span>
              <el-tooltip effect="dark" content="自然流量/Sessions*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="pageView" title="PV" min-width="100" :formatter="formatter">
            <template #header>
              <span>PV</span>
              <el-tooltip effect="dark" content="商品详情页被浏览的次数" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="spend" title="广告花费" min-width="120" :formatter="formatter">
            <template #header>
              <span>广告花费</span>
              <el-tooltip effect="dark" content="广告被展示/点击后产生的费用" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="spendRate" title="广告花费占比" min-width="140" :formatter="formatter">
            <template #header>
              <span>广告花费占比</span>
              <el-tooltip effect="dark" content="当前行广告花费/合计广告花费*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="cpc" title="CPC" min-width="120" :formatter="formatter">
            <template #header>
              <span>CPC</span>
              <el-tooltip effect="dark" content="广告花费/广告点击量" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="items" title="总订单量" min-width="120" :formatter="formatter">
            <template #header>
              <span>总订单量</span>
              <el-tooltip effect="dark" content="商品的总订单量" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="adOrders" title="广告订单量" min-width="120" :formatter="formatter">
            <template #header>
              <span>广告订单量</span>
              <el-tooltip effect="dark" content="广告带来的订单，包含广告商品和由广告带来的非广告商品的订单" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="actualOrders" title="实际广告订单量" min-width="140" sortable :formatter="formatter" />
          <vxe-table-column field="naturalOrder" title="自然订单" min-width="120" :formatter="formatter">
            <template #header>
              <span>自然订单</span>
              <el-tooltip effect="dark" content="总订单量 - 广告订单量" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="naturalOrderRate" title="自然订单占比" min-width="140" :formatter="formatter">
            <template #header>
              <span>自然订单占比</span>
              <el-tooltip effect="dark" content="自然订单/总订单量*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="adUnits" title="广告销量" min-width="120" :formatter="formatter">
            <template #header>
              <span>广告销量</span>
              <el-tooltip effect="dark" content="广告带来的销量，包含广告商品和由广告带来的非广告商品的销量" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="actualSalesVolume" title="实际广告销量" min-width="120" sortable :formatter="formatter" />
          <vxe-table-column field="orders" title="总销量" min-width="120" :formatter="formatter">
            <template #header>
              <span>总销量</span>
              <el-tooltip effect="dark" content="商品的总销量" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="adUnitsRate" title="广告销量占比" min-width="140" :formatter="formatter">
            <template #header>
              <span>广告销量占比</span>
              <el-tooltip effect="dark" content="（当前行广告销量/合计广告销量）*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="adConversion" title="广告转化率（CVR）" min-width="170" :formatter="formatter">
            <template #header>
              <span>广告转化率（CVR）</span>
              <el-tooltip effect="dark" content="广告订单量/广告点击量*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="tconversion" title="总转化率" min-width="120" :formatter="formatter">
            <template #header>
              <span>总转化率</span>
              <el-tooltip effect="dark" content="（总订单量/Sessions）*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="adSales" title="广告销售额" min-width="120" :formatter="formatter">
            <template #header>
              <span>广告销售额</span>
              <el-tooltip effect="dark" content="广告带来的订单销售额，包含广告商品和由广告带来的非广告商品的订单销售额" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="actualSales" title="实际广告销售额" min-width="150" sortable :formatter="formatter" />
          <vxe-table-column field="adSalesRate" title="广告销售额占比" min-width="150" :formatter="formatter">
            <template #header>
              <span>广告销售额占比</span>
              <el-tooltip effect="dark" content="当前行广告销售额/合计广告销售额*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="sales" title="总销售额" min-width="120" :formatter="formatter">
            <template #header>
              <span>总销售额</span>
              <el-tooltip effect="dark" content="商品的总销售额" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="salesRate" title="总销售额占比" min-width="140" :formatter="formatter">
            <template #header>
              <span>总销售额占比</span>
              <el-tooltip effect="dark" content="当前行总销售额/合计总销售额*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="acos" title="ACOS" min-width="120" :formatter="formatter">
            <template #header>
              <span>ACOS</span>
              <el-tooltip effect="dark" content="广告花费/广告销售额*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="roas" title="ROAS" min-width="120" :formatter="formatter">
            <template #header>
              <span>ROAS</span>
              <el-tooltip effect="dark" content="广告销售额/广告花费" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="gxb" title="广销比（TACOS）" min-width="160" :formatter="formatter">
            <template #header>
              <span>广销比（TACOS）</span>
              <el-tooltip effect="dark" content="广告花费/总销售额*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="asos" title="ASoAS" min-width="120" :formatter="formatter">
            <template #header>
              <span>ASoAS</span>
              <el-tooltip effect="dark" content="广告销售额/总销售额*100%" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="cpo" title="CPO" min-width="120" :formatter="formatter">
            <template #header>
              <span>CPO</span>
              <el-tooltip effect="dark" content="广告花费/广告订单量" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="avgPrice" title="平均价格" min-width="120" :formatter="formatter">
            <template #header>
              <span>平均价格</span>
              <el-tooltip effect="dark" content="总销售额/总销量" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="returnRate" title="退货率" min-width="120" :formatter="formatter" />
          <vxe-table-column field="inventory" title="FBA可售" min-width="120" :formatter="formatter" />
          <vxe-table-column field="zht" title="周转天数" min-width="120" :formatter="formatter">
            <template #header>
              <span>周转天数</span>
              <el-tooltip effect="dark" content="FBA可售/平均销量，平均销量等于，条件范围内销量/销量大于0的天数" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </template>
          </vxe-table-column>
          <vxe-table-column field="profitRate" title="利润率" min-width="120" :formatter="formatter" />
        </vxe-table>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">关闭</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  getAdvTrend, getAdvTrendExport,
} from '@/api/erp/amazon-advertise'
import * as echarts from 'echarts'
import {
  chartSelected, lineChartOption, precentDetailField, symbolDetailField,
} from './config'

export default {
  props: {
    detailRow: {
      type: Object,
      default: () => {},
    },
    visible: {
      type: Boolean,
      default: false,
    },
    saveForm: {
      type: Object,
      default: () => {},
    },
  },
  data() {
    return {
      form: {
        time: [],
        dateType: '日',
      },
      formRules: {},
      searchLoading: false,
      tableData: [],
      maxHeight: 500,
      xTableRef: null,
      footerData: [],
      exportLoading: false,
      data: {},

      chartData: {
        xData: [],
        yData: [],
        selected: {},
      },
      chartSelected: chartSelected,
      chart: null,
      showSelected: [],
      chartOption: {
        dataZoom: [{}],
        grid: {
          top: '150',
          left: '80',
          containLabel: false,
        },
        legend: {
          selected: {},
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
          },
        },
        xAxis: {
          type: 'category',
          data: [],
          boundaryGap: false,
        },
        yAxis: [],
        series: [],
      },
    }
  },
  computed: {
    dateTypeMap() {
      return [
        { label: '日', value: 'day' },
        { label: '周', value: 'week' },
        { label: '月', value: 'month' },
      ]
    },
    typeMap() {
      return {
        'CT': 'combatTeam',
        'category': 'categoryId',
        'style': 'styleId',
      }
    },
    queryParams() {
      const [startDate, endDate] = this.form.time || []
      const params = {
        ...this.saveForm,
        ...this.form,
        startDate, endDate,
        dateType: this.dateTypeMap.find(item => item.label === this.form.dateType)?.value,
      }
      if (this.detailRow.dataType === 'CT') {
        params[`${this.typeMap[this.detailRow.dataType]}List`] =
        this.detailRow?.[`${this.typeMap[this.detailRow.dataType]}Key`]
          ? [this.detailRow[`${this.typeMap[this.detailRow.dataType]}Key`]]
          : []
      } else {
        params[`${this.typeMap[this.detailRow.dataType]}List`] =
        this.detailRow?.[this.typeMap[this.detailRow.dataType]]
          ? [this.detailRow[this.typeMap[this.detailRow.dataType]]]
          : []
      }
      return params
    },
    dialogHeight() {
      return window.innerHeight - 300
    },
  },
  watch: {
    visible: {
      handler(val) {
        this.$nextTick(() => {
          if (val) {
            this.xTableRef = this.$refs?.['table-dialog-ref']
            this.form.time = this.saveForm?.time || []
            this.initChart()
            this.getData()
          }
        })
      },
    },
  },
  mounted() {
  },
  destroyed() {
    this.destroyChart()
  },
  methods: {
    async handleExport() {
      this.exportLoading = true
      await getAdvTrendExport(this.queryParams).finally(() => {
        this.exportLoading = false
      })
    },
    async getData() {
      this.chart?.showLoading()
      this.searchLoading = true
      const { datas = {}} = await getAdvTrend(this.queryParams).finally(() => {
        this.chart?.hideLoading()
        this.searchLoading = false
      })
      this.data = datas
      this.footerData = []
      this.tableData = this.data?.detailDataList || []
      this.footerData.push({
        ...this.formatterFooter(this.data?.totalData),
        categoryName: '合计',
        combatTeam: '合计',
        styleName: '合计',
      })
      this.assemblyData()
    },
    formatterFooter(total) {
      const totalObj = {}
      Object.entries(total).forEach(([key, value]) => {
        totalObj[key] = this.formatter({ cellValue: value, column: { property: key }, row: total })
      })
      return totalObj
    },
    assemblyData(chartSelected) {
      this.chartSelected = chartSelected || this.chartSelected
      Object.keys(this.chartSelected).forEach((item) => {
        if (!this.showSelected.includes(item) && this.chartSelected[item]) {
          this.showSelected.push(item)
        }
        if (this.showSelected.includes(item) && !this.chartSelected[item]) {
          this.showSelected = this.showSelected.filter(i => i !== item)
        }
      })
      // 最长只能四个
      if (this.showSelected.length > 4) {
        const name = this.showSelected[0]
        this.showSelected.shift()
        this.chartSelected[name] = false
      }

      this.chartData.yData = []
      // y轴数据
      let index = 0
      lineChartOption.forEach((item) => {
        const name = item.name
        const obj = {
          name: name,
          type: 'line',
          data: this.data?.trafficTrend?.[item.yAxisfield] || [],
          yAxisIndex: this.chartSelected[name] ? index : 0,
        }
        if (this.chartSelected[name]) {
          index++
        }
        this.chartData.yData.push(obj)
      })
      // y轴
      let i = 0
      const yAxis = []
      this.chartData.yData.forEach((item) => {
        const name = item.name
        if (this.chartSelected[name]) {
          const obj = {
            type: 'value',
            name: name,
            axisLine: { show: true },
            axisTick: { show: true },
            splitLine: { show: false },
            offset: i > 1 ? 100 * (i - 1) : 0,
          }
          if ([
            '广告点击率（CTR）',
            '自然流量占比',
            '广告花费占比',
            '自然订单占比',
            '广告销量占比',
            '广告转化率（CVR）',
            '总转化率',
            '广告销售额占比',
            '总销售额占比',
            'ACOS',
            '广销比（TACOS）',
            'ASoAS',
            '退货率',
            '利润率',
          ].includes(name)) {
            obj.axisLabel = {
              formatter: '{value}%',
            }
          }
          i++
          yAxis.push(obj)
        }
      })
      this.chartData.yAxis = yAxis
      this.chartData.selected = this.chartSelected
      this.chartData.xData = this.data?.trafficTrend?.dateList

      this.setChartData()
    },
    cancel() {
      this.chart?.clear()
      this.xTableRef?.clearSort()
      this.$emit('update:visible', false)
    },
    initChart() {
      if (!this.chart) {
        this.chart = echarts.init(this.$refs['chart'])
        this.chart.on('legendselectchanged', (params) => {
          // 最后一个不允许取消
          if (Object.values(params.selected).filter(value => value).length === 0) {
            params.selected[params.name] = true
          }
          this.assemblyData(params.selected)
        })
        window.addEventListener('resize', this.chartResize)
      }
    },
    setChartData() {
      this.chart?.clear()
      this.chartOption.series = this.chartData.yData
      this.chartOption.xAxis.data = this.chartData.xData
      this.chartOption.yAxis = this.chartData.yAxis
      this.chartOption.grid.right = ((this.chartData.yAxis.length - 1) * 100)
      this.chartOption.legend.selected = this.chartData?.selected || {}
      this.chart.setOption(this.chartOption)
    },
    destroyChart() {
      window.removeEventListener('resize', this.chartResize)
      this.chart?.off('legendselectchanged')
      this.chart?.dispose()
      this.chart = null
    },
    chartResize() {
      this.chart && this.chart.resize()
    },
    formatter({ cellValue, column, row }) {
      const cellValueNumber = +cellValue
      const cellValueNumberToLocaleString = cellValueNumber?.toLocaleString()
      if (cellValueNumber && precentDetailField.includes(column.property)) {
        return `${cellValueNumberToLocaleString} %`
      } else if (cellValueNumber && symbolDetailField.includes(column.property)) {
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

<style lang="scss" scoped>

</style>
