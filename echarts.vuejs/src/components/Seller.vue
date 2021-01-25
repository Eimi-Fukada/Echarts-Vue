<template>
  <div class="com-container">
      <div class="com-chart" ref="seller_ref"></div>
  </div>
</template>

<script>
export default {
  data (){
    return {
      chartInstance: null
    }
  },
  mounted () {
    this.initChart()
    this.getData()
  },
  methods: {
    // 初始化echartInstance对象
    initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.seller_ref)
    },
    // 获取服务器数据
    async getData () {
      const ret = await this.$http.get('seller')
      console.log(ret)
    },
    // 更新图表
    updataChart () {

    },
  }
}
</script>

<style lang="less" scoped>
</style>
