<template>
  <div class="com-container" >
      <div class="com-chart" ref="rank_ref"></div>
  </div>
</template>

<script>
export default {
    data (){
        return {
            chartInstance: null,
            allData: null, //服务器返回的数据
            startValue: 0,
            endValue: 9,
            timeId: null
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
            this.chartInstance = this.$echarts.init(this.$refs.rank_ref,'chalk')
            const initOption = {
                title: {
                    text: '▎地区销售排行', 
                    left:20,
                    top:20,
                },
                grid: {
                    top:'40%',
                    left:'5%',
                    right:'5%',
                    bottom:'5%',
                    containLabel:true,
                },
                tooltip: {
                    trigger: 'axis',
                    axisPointer: {
                        type: 'line',
                        z:0,
                        lineStyle: {
                        color: '#2D3443'
                        }
                    }
                },
                xAxis: {
                    type: 'category'
                },
                yAxis: {
                    type: 'value'
                },
                series: [
                    {
                        type:'bar'
                    }
                ]
            }
            this.chartInstance.setOption(initOption)
            // 对图表对象进行鼠标移入移出的监听
            this.chartInstance.on('mouseover', () => {
                clearInterval(this.timeId)
            })
            this.chartInstance.on('mouseout', () => {
                this.startInterval()
            })
        },

        async getData () {
            const { data: ret } = await this.$http.get('rank')
            this.allData = ret
            this.allData.sort((a,b) => {
                return b.value-a.value
            })
            this.updataChart()
            this.startInterval()
        }, 

        updataChart () {
            const colorArr = [ 
                ['#0BA82C', '#4FF778'], 
                ['#2E72BF', '#23E5E5'], 
                ['#5052EE', '#AB6EE5'] 
            ]
            const provinceArr = this.allData.map(item => {
                return item.name
            })
            const valueArr = this.allData.map(item => {
                return item.value
            })
            const dataOption = {
                xAxis: {
                    data: provinceArr
                },
                dataZoom: {
                    show: false,
                    startValue: this.startValue,
                    endValue: this.endValue,
                },
                series: [
                    {
                        data:valueArr,
                        itemStyle: {
                            color: (arg) => {
                                let targetColorArr = colorArr[0]
                                if (arg.value > 300) {
                                    targetColorArr = colorArr[0]
                                } else if (arg.value > 200) {
                                    targetColorArr = colorArr[1]
                                } else {
                                    targetColorArr = colorArr[2]
                                }
                                return new this.$echarts.graphic.LinearGradient(0,0,0,1,[
                                    {
                                        offset: 0, 
                                        color: targetColorArr[0]
                                    },
                                    {
                                        offset: 1, 
                                        color: targetColorArr[1]
                                    }
                                ])
                            },
                        },
                    }
                ]
            }
            this.chartInstance.setOption(dataOption)
        },

        screenAdapter() {
            const titleFontSize = this.$refs.rank_ref.offsetWidth / 100 * 3.6
            const adapterOption = {
                title: {
                    textStyle: {
                        fontSize: titleFontSize
                    }
                },
                 series: [
                    {           
                        barWidth: titleFontSize,
                        itemStyle: {
                        barBorderRadius: [titleFontSize / 2,titleFontSize / 2,0,0],
                        },
                    }
                ]
            }
            this.chartInstance.setOption(adapterOption)
            this.chartInstance.resize()
        },

        startInterval() {
            if (this.timeId) {
                clearInterval(this.timeId)
            }
            this.timeId = setInterval(() => {
                this.startValue++
                this.endValue++
                if (this.endValue > this.allData.length-1) {
                    this.startValue = 0
                    this.endValue = 9
                }
                this.updataChart()
            },2000)
        },

    },


}
</script>

<style lang="less" scoped>
</style>