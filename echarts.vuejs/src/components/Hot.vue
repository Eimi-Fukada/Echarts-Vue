<template>
  <div class="com-container" >
      <div class="com-chart" ref="hot_ref"></div>
  </div>
</template>

<script>
export default {
    data (){
        return {
            chartInstance: null,
            allData: null, //服务器返回的数据
        }
    },

    mounted () {
        this.initChart()
        this.getData()
        window.addEventListener('resize', this.screenAdapter)
        // 页面加载完成时主动对屏幕适配
        this.screenAdapter()
    },

    destroyed () {
        window.removeEventListener('resize', this.screenAdapter)
        clearInterval(this.timeId)
    },

    methods: {
        initChart () {
            this.chartInstance = this.$echarts.init(this.$refs.hot_ref,'chalk')
            const initOption = {
                // title: {
                //     text: '▎地区销售排行', 
                //     left:20,
                //     top:20,
                // },
                // grid: {
                //     top:'40%',
                //     left:'5%',
                //     right:'5%',
                //     bottom:'5%',
                //     containLabel:true,
                // },
                // tooltip: {
                //     trigger: 'axis',
                //     axisPointer: {
                //         type: 'line',
                //         z:0,
                //         lineStyle: {
                //         color: '#2D3443'
                //         }
                //     }
                // },
                // xAxis: {
                //     type: 'category'
                // },
                // yAxis: {
                //     type: 'value'
                // },
                // series: [
                //     {
                //         type:'bar'
                //     }
                // ]
            }
            this.chartInstance.setOption(initOption)
        },

        async getData () {
            const { data: ret } = await this.$http.get('hot')
            this.allData = ret
            console.log(this.allData)
            this.updataChart()
        }, 

        updataChart () {
        
            const dataOption = {
               
            }
            this.chartInstance.setOption(dataOption)
        },

        screenAdapter() {
            // const titleFontSize = this.$refs.rank_ref.offsetWidth / 100 * 3.6
            const adapterOption = {
                // title: {
                //     textStyle: {
                //         fontSize: titleFontSize
                //     }
                // },
                //  series: [
                //     {           
                //         barWidth: titleFontSize,
                //         itemStyle: {
                //         barBorderRadius: [titleFontSize / 2,titleFontSize / 2,0,0],
                //         },
                //     }
                // ]
            }
            this.chartInstance.setOption(adapterOption)
            this.chartInstance.resize()
        },

        

    },
}
</script>

<style lang="less" scoped>
</style>