# 基于腾讯地图+Ant-Design-Vue封装省市区联动查询组件

## 一、腾讯地图key申请

附上教程：[腾讯地图key申请](https://lbs.qq.com/webApi/javascriptGL/glGuide/glBasic)

## 二、在项目加载JS API

在VUE项目的`pubilc`文件夹下的`index.html`中加入

```html
<script src="https://map.qq.com/api/gljs?v=1.exp&key=你申请的key"></script>
```

## 三、组件封装

1、在`src>components`文件夹下新建`MyMap.vue`

2、代码如下：

```vue
<template>
  <div>
    <div class="c-tmap-select">
      <a-select
        v-model="provinceValue"
        placeholder="省"
        class="c-amap_select"
      >
        <a-select-option
          v-for="item in provinceOptions"
          :key="item.id"
          :value="item.fullname"
          @click="provinceChange(item)"
        >
          {{ item.fullname }}
        </a-select-option>
      </a-select>
      <a-select
        v-model="cityValue"
        placeholder="市"
        class="c-amap_select"
      >
        <a-select-option
          v-for="item in cityOptions"
          :key="item.id"
          :value="item.fullname"
          @click="cityChange(item)"
        >
          {{ item.fullname }}
        </a-select-option>
      </a-select>
      <a-select
        v-model="areaValue"
        placeholder="区"
        class="c-amap_select"
      >
        <a-select-option
          v-for="item in areaOptions"
          :key="item.id"
          :value="item.fullname"
          @click="areaChange(item)"
        >
          {{ item.fullname }}
        </a-select-option>
      </a-select>
    </div>
    <slot></slot>
    <div id="container" style="width:800px;height:400px;">
      <div
        class="c-tmap-input"
        @click.stop="()=>{}"
        @touch.stop="()=>{}"
        @dblclick.stop="()=>{}"
      >
        <a-auto-complete
          v-model="address"
          date-source="dataSource"
          placeholder="请输入详细地址"
          class="c-tmap-input_auto"
          @change="handleChange"
        >
          <template slot="dataSource">
            <a-select-option
              v-for="item in dataSource"
              :key="item.id"
              :title="item.title"
              @click="selctLocation(item)"
            >
              {{ item.title }}--{{ item.address }}
            </a-select-option>
          </template>
        </a-auto-complete>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MyMap',
  props: {
    // 省
    provinceProp: {
      type: String,
      default: ''
    },
    // 市
    cityProp: {
      type: String,
      default: ''
    },
    // 区
    areaProp: {
      type: String,
      default: ''
    },
    // 经度
    longitude: {
      type: String,
      default: ''
    },
    // 纬度
    latitude: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      markerLayer: {},
      map: {},
      lng: 116.40717, // 经度
      lat: 39.90469, // 纬度,
      address: '', // 详细地址
      dataSource: [], // 详细地址输入提示
      provinceValue: '', // 省
      cityValue: '', // 市
      areaValue: '', // 区
      provinceOptions: [], // 省级行政区选择
      cityOptions: [], // 市级行政区选择
      areaOptions: [], // 县区行政区选择
      provinces: [], // 省级数据
      cities: [], //  市级数据
      areas: [] // 地区数据
    }
  },
  async mounted() {
    this.init()
    this.markerLayer = new window.TMap.MultiMarker({})
    this.initMap()
    this.addImgMarker(new window.TMap.LatLng(this.lat, this.lng))
    await this.getDistrict()

  },
  methods: {
    // 省市区数据初始化
    init() {
      if (this.provinceProp) {
        this.provinceValue = this.provinceProp
      }
      if (this.cityProp) {
        this.cityValue = this.cityProp
      }
      if (this.areaProp) {
        this.areaValue = this.areaProp
      }
    },
    // 重置市、区、详细地址数据
    resetCityAndareaData() {
      this.cityValue = ''
      this.areaValue = ''
      this.address = ''
      this.cityOptions = []
      this.areaOptions = []
    },
    // 重置区、详细地址数据
    resetareaData() {
      this.areaValue = ''
      this.address = ''
      this.areaOptions = []
    },
    // 选择省级地区触发
    provinceChange(item) {
      this.resetCityAndareaData()
      // console.log(item.cidx)
      if (item && item.cidx) {
        this.cityOptions = this.cities.slice(item.cidx[0], item.cidx[1] + 1)
        this.getLocation(item.location)
      }
      this.provinceValue = item.fullname
      this.chooseData()
      // console.log(item.cidx[0], item.cidx[1] + 1)

    },
    // 选择城市触发
    cityChange(item) {
      this.resetareaData()
      // console.log(item)
      if (item && item.cidx) {
        this.areaOptions = this.areas.slice(item.cidx[0], item.cidx[1] + 1)
        this.getLocation(item.location)
      }
      this.cityValue = item.fullname
      // console.log(item.cidx[0], item.cidx[1] + 1)
      // console.log(this.areaOptions)
      this.chooseData()
    },
    // 选择地区触发
    areaChange(item) {
      this.areaValue = item.fullname
      this.chooseData()
      if (item) {
        this.getLocation(item.location)
      }
    },
    chooseData() {
      // console.log(this.provinceValue, this.cityValue, this.areaValue, this.address)
      const data = {
        provinceValue: this.provinceValue,
        cityValue: this.cityValue,
        areaValue: this.areaValue,
        address: this.address
      }
      this.$emit('change', data)
    },
    // 获取全部行政区划数据
    getDistrict() {
      this.$store.dispatch('map/getDistrict').then(res => {
        this.district = res.result
        this.provinces = res.result[0]
        this.provinceOptions = res.result[0]
        this.cities = res.result[1]
        this.areas = res.result[2]
        // console.log(this.district)
      })
    },
    selctLocation(item) {
      // console.log(item)
      this.getLocation(item.location)
      this.address = item.title + '-' + item.address
      this.chooseData()
    },
    // 获取当前位置，并在地图中显示
    getLocation(location) {
      // console.log('location', location)
      this.lat = location.lat
      this.lng = location.lng
      this.map.setCenter(new window.TMap.LatLng(this.lat, this.lng))
      this.addImgMarker(new window.TMap.LatLng(this.lat, this.lng))
    },
    // 选择搜索
    handleChange() {
      const that = this
      this.$store.dispatch('map/getSuggestion', {
        region: this.cityValue || '',
        keyword: this.address,
      }).then(res => {
        that.dataSource = res.data
      })
    },
    // 地图初始化
    initMap() {
      // console.log('window', window)
      const that = this
      // 定义地图中心点坐标
      // eslint-disable-next-line no-undef
      const center = new TMap.LatLng(this.lat, this.lng)
      // 定义map变量，调用 TMap.Map() 构造函数创建地图
      // eslint-disable-next-line no-undef
      const map = new TMap.Map(document.getElementById('container'), {
        center: center, // 设置地图中心点坐标
        zoom: 15, // 设置地图缩放级别
        pitch: 43.5, // 设置俯仰角
        rotation: 45, // 设置地图旋转角度
      })
      this.map = map
      map.on('click', (e) => {
        this.markerLayer.setGeometries([])
        const position = e.latLng
        // console.log(e)
        // console.log(position.lat.toFixed(6))
        this.$store.dispatch('map/getClickLocation', {
          lat: position.lat.toFixed(6),
          lng: position.lng.toFixed(6)
        }).then(res => {
          // console.log(res)
          that.provinceValue = res.result.address_component.province
          that.cityValue = res.result.address_component.city
          that.areaValue = res.result.address_component.district
          that.address = res.result.formatted_addresses.recommend
          that.chooseData()
        })
        that.addImgMarker(position)
      })
    },
    // 在地图上添加点标识
    addImgMarker(data) {
      // console.log(data, 'data')
      this.markerLayer = new window.TMap.MultiMarker({
        map: this.map,
        // 样式定义
        styles: {
        // 创建一个styleId为"myStyle"的样式（styles的子属性名即为styleId）
          myStyle: new window.TMap.MarkerStyle({
            width: 25, // 点标记样式宽度（像素）
            height: 35, // 点标记样式高度（像素）
            anchor: { x: 16, y: 32 },
          }),
        },
        // 点标记数据数组
        geometries: [
          {
            id: '1', // 点标记唯一标识，后续如果有删除、修改位置等操作，都需要此id
            styleId: 'myStyle', // 指定样式id
            position: data, // 点标记坐标位置
          },
        ],
      })
    },
  },

}
</script>

<style lang="scss">
.c-tmap-select {
  margin: 10px;
}
.c-tmap-input {
  position: absolute;
  z-index: 9999;
  top: 10px;
  right: 85px;
}
.c-tmap-input_auto {
  width: 300px;
}
</style>

```

## 四、效果

![image-20211103151845625](https://raw.githubusercontent.com/Pizhong/PicGoImg/main/image-20211103151845625.png)