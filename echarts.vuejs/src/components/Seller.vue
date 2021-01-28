<template>
  <div class="com-container">
      <div class="com-chart" ref="seller_ref"></div>
  </div>
</template>

<script>
export default {
  data (){
    return {
      chartInstance: null,
      allData: null, //服务器返回的数据
      currentPages: 1, //当前显示的页数
      totalPage: 0,  //总共多少页
      timerId: null,  //定时器的标识
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
    clearInterval(this.timerId)
    window.removeEventListener('resize', this.screenAdapter)
  },
  methods: {

    // 初始化echartInstance对象
    initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.seller_ref,'chalk')

      // 对图表初始化配置的控制
      const initOption = {
        title: {
          text: '▎商品销售统计', 
          left:80,
          top:40,
        },
        grid: {
          top:'20%',
          left:'3%',
          right:'6%',
          bottom:'3%',
          containLabel:true,   //距离是包含坐标轴上的文字
        },
        xAxis: {
          type: 'value'
        },
        yAxis: {
          type: 'category',
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
        series: [
          {
            type: 'bar',          
            label: {
              show: true,
              position: 'right',
              textStyle: {
                color: 'white'
              },
            },
            itemStyle: {
              // 指明颜色渐变的方向
              // 指明不同百分比下颜色的值
              color: new this.$echarts.graphic.LinearGradient(0, 0, 1, 0, [
                // 0%状态下的颜色
                {
                  offset: 0, 
                  color: '#5052EE'
                },
                // 100%状态下的颜色
                {
                  offset: 1, 
                  color: '#AB6EE5'
                }
              ])
            },
          }
        ]
      }
      this.chartInstance.setOption(initOption)

      // 对图表对象进行鼠标移入移出的监听
      this.chartInstance.on('mouseover', () => {
        clearInterval(this.timerId)
      })
      this.chartInstance.on('mouseout', () => {
        this.startInterval()
      })
    },

    // 获取服务器数据
    async getData () {
      // http://127.0.0.1:8888/api/seller
      const { data: ret } = await this.$http.get('seller')
      this.allData = ret
      // 排序
      this.allData.sort((a,b) => {
        // 排序规则是根据返回值确定的，小于0则升序，大于0则降序
        return a.value-b.value   
      })
      this.totalPage = this.allData.length % 5 ===0 ? this.allData.length / 5 : this.allData.length / 5 + 1
      this.updataChart()
      this.startInterval()
    },

    // 更新图表
    updataChart () {
      const start = (this.currentPages - 1)*5
      const end =  this.currentPages * 5 
      // 截取后台数据的一部分展示
      const showData = this.allData.slice(start,end)
      const sellerName = showData.map((item) => {
        return item.name
      })
      const sellerValue = showData.map((item) => {
        return item.value
      })
      const dataOption = {
        yAxis: {
          data: sellerName
        },     
        series: [
          {
            data: sellerValue,
          }
        ]
      }
      this.chartInstance.setOption(dataOption)
    },

    // 定时器
    startInterval () {
      if (this.timerId) {
        clearInterval(this.timerId)
      }
      this.timerId = setInterval(() => {
        this.currentPages++
        if (this.currentPages > this.totalPage) {
          this.currentPages = 1
        }
        this.updataChart()
      },3000)
    },

    // 当浏览器大小变化时调用
    screenAdapter() {
      const titleFontSize = this.$refs.seller_ref.offsetWidth / 100 * 3.6
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: titleFontSize
          },
        },
        tooltip: {
          axisPointer: {
            lineStyle: {
              width: titleFontSize, 
            }
          }
        },
        series: [
          {           
            barWidth: titleFontSize,
            itemStyle: {
              barBorderRadius: [0,titleFontSize / 2,titleFontSize / 2,0],
            },
          }
        ]
      }
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    },


  }
}
</script>

<style lang="less" scoped>
</style>
