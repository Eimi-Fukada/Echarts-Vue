<template>
  <div class="com-container" >
      <div class="com-chart" ref="hot_ref"></div>
      <span class=""></span>
      <span class=""></span>
  </div>
</template>

<script>
export default {
    data (){
        return {
            chartInstance: null,
            allData: null, //服务器返回的数据
            currentIndex:0,
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
                series:[
                    {
                        type:'pie',
                    }
                ]
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
            const legendData = this.allData[this.currentIndex].children.map(item => {
                return item.name
            })
            const seriesData = this.allData[this.currentIndex].children.map(item => {
                return {
                    name:item.name,
                    value:item.value
                }
            })
            const dataOption = {
                legend:{
                    data:legendData
                },
                series:[
                    {
                        data:seriesData
                    }
                ]
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