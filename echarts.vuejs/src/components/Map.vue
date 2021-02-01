<template>
  <div class="com-container" @dblclick="revertMap">
      <div class="com-chart" ref="map_ref"></div>
  </div>
</template>

<script>
import axios from 'axios'
import { getProvinceMapInfo } from '../utils/map_utils'
export default {
    data (){
        return {
            chartInstance: null,
            allData: null, //服务器返回的数据
            mapData: {}  //所获取的省份的地图矢量数据
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

    methods: {
        async initChart () {
            this.chartInstance = this.$echarts.init(this.$refs.map_ref,'chalk')
            const ret = await axios.get('http://localhost:8999/static/map/china.json')
            this.$echarts.registerMap('china',ret.data)
            // console.log(ret)
            // 获取中国地图的矢量数据
            const initOption = {
                title: {
                    text: '▎商家分布',
                    left: 20,
                    top:20
                },
                geo: {
                    type: 'map',
                    map: 'china',
                    top:'5%',
                    bottom: '5%',
                    itemStyle: { 
                        areaColor: '#2E72BF', 
                        borderColor: '#333' 
                    }
                },
                legend: { 
                    left: '5%', 
                    bottom: '5%', 
                    orient: 'vertical' 
                }
            }
            this.chartInstance.setOption(initOption)
            this.chartInstance.on('click',async arg=>{
                const provinceInfo = getProvinceMapInfo(arg.name)
                // 判断当前所点的这个省份的地图矢量数据在mapdata里有没有
                if (!this.mapData[provinceInfo.key]) {
                    const ret = await axios.get('http://localhost:8999' + provinceInfo.path)
                    console.log(ret)
                    this.mapData[provinceInfo.key] = ret.data    //缓存数据
                    this.$echarts.registerMap(provinceInfo.key, ret.data)
                }
                const changeOption = {
                    geo: {
                        map: provinceInfo.key
                    }
                }
                this.chartInstance.setOption(changeOption)
            })
        },

        // 获取服务器数据
        async getData () {
            const { data: ret } = await this.$http.get('map')
            this.allData = ret
            // console.log(this.allData)
            this.updataChart()
        },

        // 更新图表
        updataChart () {
            // 图例数据
            const legendArr = this.allData.map(item => {
                return item.name
            })
            // 图表数据
            const seriesArr = this.allData.map(item => {
                // return返回的对象就是一个类别下的所有散点数据
                return {
                    type: 'effectScatter', 
                    rippleEffect: { 
                        scale: 5, 
                        brushType: 'stroke' 
                    },
                    name: item.name, 
                    data: item.children,
                    coordinateSystem: 'geo'
                }
            })
            const dataOption = {
                legend: {
                    data: legendArr
                },
                series: seriesArr
            }
            this.chartInstance.setOption(dataOption)
        },

        // 当浏览器大小变化时调用
        screenAdapter() {
            const titleFontSize = this.$refs.map_ref.offsetWidth / 100 * 3.6
            const adapterOption = {
                title: {
                    textStyle: {
                        fontSize: titleFontSize
                    }
                },
                legend: {
                    itemWidth: titleFontSize / 2, 
                    itemHeight: titleFontSize / 2, 
                    itemGap: titleFontSize / 2,
                    textStyle: {
                        fontSize: titleFontSize / 2
                    }
                }
            }
            this.chartInstance.setOption(adapterOption)
            this.chartInstance.resize()
        },

        // 回到中国地图
        revertMap () {
            const revertOption = {
                geo: {
                    map: 'china'
                }
            }
            this.chartInstance.setOption(revertOption)
        },

    },

}
</script>

<style lang="less" scoped>
</style>