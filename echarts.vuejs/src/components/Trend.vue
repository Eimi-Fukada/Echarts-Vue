<template>
  <div class="com-container">
      <div class="title">
          <span>我是标题</span>
          <span class="iconfont title-icon" @click="showChoice = !showChoice">&#xe6eb;</span>
          <div class="select-con" v-show="showChoice">
              <div class="select-item" v-for="item in selectTypes" :key="item.key">
                  {{item.text}}
              </div>
          </div>
      </div>
      <div class="com-chart" ref="trend_ref"></div>
  </div>
</template>

<script>
export default {
  data (){
    return {
      chartInstance: null,
      allData: null, //服务器返回的数据
      showChoice:false  //是否显示切换项
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
  },

  computed: {
      selectTypes () {
          if (!this.allData) {
              return []
          }else {
              return this.allData.type
          }
      }
  },

  methods: {
    // 初始化echartInstance对象
    initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.trend_ref,'chalk')
      // 对图表初始化配置的控制
      const initOption = {
        // title: {
        //   text: '▎商品销售统计', 
        //   left:80,
        //   top:40,
        // },
        grid: {
          top:'20%',
          left:'3%',
          right:'6%',
          bottom:'3%',
          containLabel:true,   //距离是包含坐标轴上的文字
        },
        xAxis: {
          type: 'category',
          boundaryGap: false
        },
        yAxis: {
          type: 'value',
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
        legend: {
          top: '12%',
          left: 80 ,
          icon: 'circle'
        },
        
      }
      this.chartInstance.setOption(initOption)
    },

    // 获取服务器数据
    async getData () {
      const { data: ret } = await this.$http.get('trend')
      this.allData = ret
      this.updataChart()
      
    },

    // 更新图表
    updataChart () {
        // 半透明颜色值
      const colorArr1 = [ 
          'rgba(11, 168, 44, 0.5)', 
          'rgba(44, 110, 255, 0.5)', 
          'rgba(22, 242, 217, 0.5)', 
          'rgba(254, 33, 30, 0.5)', 
          'rgba(250, 105, 0, 0.5)' 
      ]
        //   全透明颜色值
      const colorArr2 = [
          'rgba(11, 168, 44, 0)', 
          'rgba(44, 110, 255, 0)', 
          'rgba(22, 242, 217, 0)', 
          'rgba(254, 33, 30, 0)', 
          'rgba(250, 105, 0, 0)' 
      ]
      const timeArr = this.allData.common.month
      const valueArr = this.allData.map.data
      const seriesArr = valueArr.map((item,index) => {
          return {
              type: 'line',
              name: item.name,
              data: item.data,
              stack: 'map',
              areaStyle: {
                  color: new this.$echarts.graphic.LinearGradient(0, 0, 0, 1, [
                    // 0%状态下的颜色
                    {
                    offset: 0, 
                    color: colorArr1[index]
                    },
                    // 100%状态下的颜色
                    {
                    offset: 1, 
                    color: colorArr2[index]
                    }
                ])
              }
          }
      })
    //   图例数据：上海，北京。。。
      const legendArr = valueArr.map(item => {
          return item.name
      })
      const dataOption = {
        xAxis: {
          data: timeArr
        },   
        legend: {
          data: legendArr
        },  
        series:  seriesArr,

      }
      this.chartInstance.setOption(dataOption)
    },

        // 当浏览器大小变化时调用
    screenAdapter() {
    //   const titleFontSize = this.$refs.trend_ref.offsetWidth / 100 * 3.6
      const adapterOption = {
        // title: {
        //   textStyle: {
        //     fontSize: titleFontSize
        //   },
        // },
        // tooltip: {
        //   axisPointer: {
        //     lineStyle: {
        //       width: titleFontSize, 
        //     }
        //   }
        // },
        // series: [
        //   {           
        //     barWidth: titleFontSize,
        //     itemStyle: {
        //       barBorderRadius: [0,titleFontSize / 2,titleFontSize / 2,0],
        //     },
        //   }
        // ]
      }
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    },


  },

}
</script>

<style lang="less" scoped>
.title{
    position: absolute;
    left: 20px;
    top: 20px;
    z-index: 10;
    color: #fff;
    .title-icon{
        margin-left: 10px;
        cursor: pointer;
    }
    .select-item{
        cursor: pointer;
    }
}
</style>