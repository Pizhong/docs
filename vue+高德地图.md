# 基于高德地图+Ant-Design-Vue封装省市区联动查询组件

## 一、高德地图key申请

1. 首先，[注册开发者账号](https://lbs.amap.com/dev/id/newuser)，成为高德开放平台开发者

2. 登陆之后，在进入「应用管理」 页面「创建新应用」

3. 为应用[添加 Key](https://lbs.amap.com/dev/key/app)，「服务平台」一项请选择「 Web 端 ( JSAPI ) 」

   

## 二、在项目加载JS API

在VUE项目的`pubilc`文件夹下的`index.html`中加入

```html
<script type="text/javascript" src="https://webapi.amap.com/maps?v=1.4.15&key=您申请的key值&plugin=AMap.DistrictSearch"></script> 
```



## 三、在 vue.config.js 文件中配置 externals

1、在项目根目录新建vue.config.js文件

2、加入下面配置

```javascript
module.exports = {
  configureWebpack: {
    externals: {
      'AMap': 'AMap' // 高德地图配置
   } 
  }
}
```



## 四、组件封装

1、在`src>components`文件夹下新建`MyMap.vue`

2、代码如下：

```vue
<template>
  <div class="mapContent">
    <div id="tip">
      <a-select
        v-model="province"
        placeholder="省"
        class="c-amap_select"
        @change="selectP"
      >
        <a-select-option
          v-for="item in provinces"
          :key="item.adcode"
          :value="item.adcode"
        >
          {{ item.name }}
        </a-select-option>
      </a-select>
      <a-select
        v-model="city"
        placeholder="市"
        class="c-amap_select"
        @change="selectC"
      >
        <a-select-option
          v-for="item in citys"
          :key="item.adcode"
          :value="item.name"
        >
          {{ item.name }}
        </a-select-option>
      </a-select>
      <a-select
        v-model="district"
        placeholder="区"
        class="c-amap_select"
        @change="selectD"
      >
        <a-select-option
          v-for="item in districts"
          :key="item.adcode"
          :value="item.adcode"
        >
          {{ item.name }}
        </a-select-option>
      </a-select>
      <a-input
        id="searchValue"
        v-model="address"
        placeholder="地址"
        style="margin:10px 0"
      />
    </div>
    <div id="mapContainer"></div>
  </div>
</template>

<script>
import AMap from 'AMap' // 引入高德地图
export default {
  name: 'MyMap',
  props: ['position'], // 经纬度
  data() {
    return {
      map: null,
      provinces: [],
      province: '',
      districts: [],
      district: '',
      citys: [],
      city: '',
      polygons: [],
      areacode: '',
      address: '',
      objPoint: [],
      adcode: '全国',
      center: [],
      // position: []
    }
  },
  watch: {
    adcode: {
      handler: function() {
        this.loadSearch()
      }
    }
  },
  mounted() {
    setTimeout(() => {
      this.initMap()
      this.loadSearch()
    }, 0)
    this.loadmap()
  },
  methods: {
    // 地图初始化
    initMap() {
      const that = this
      if (this.center.length > 0) {
        this.map = new AMap.Map('mapContainer', {
          resizeEnable: true,
          zoom: 11,
          center: that.center
        })
      } else {
        this.map = new AMap.Map('mapContainer', {
          resizeEnable: true,
          zoom: 11
        })
      }
      this.marker = new AMap.Marker({
        icon: new AMap.Icon({
          size: new AMap.Size(36, 44),
        }),
        map: this.map
      })
    },
    // 输入提示与POI搜索
    loadSearch() {
      const that = this
      AMap.plugin(['AMap.Autocomplete', 'AMap.PlaceSearch', 'AMap.Geocoder'], function() {
        const autocomplete = new AMap.Autocomplete({
          city: that.adcode, // 城市，默认全国
          input: 'searchValue'
        })
        const placeSearch = new AMap.PlaceSearch({
          city: '全国',
          citylimit: true,
          pageSize: 1,
          pageIndex: 1,
          map: that.map
        })
        const geocoder = new AMap.Geocoder({
          radius: 1000
        })
        AMap.event.addListener(autocomplete, 'select', function(e) {
          that.marker.setMap(that.map)
          that.marker.setPosition(e.poi.location)
          placeSearch.search()// 关键字查询查询
          geocoder.getAddress(e.poi.location, function(status, result) {
            if (status === 'complete') {
              const obj = {}
              that.province = result.regeocode.addressComponent.province
              that.city = result.regeocode.addressComponent.city
              that.district = result.regeocode.addressComponent.district
              obj.formattedAddress = result.regeocode.formattedAddress
              obj.areacode = result.regeocode.addressComponent.adcode
              that.address = result.regeocode.formattedAddress
              obj.lat = e.poi.location.lat
              obj.lng = e.poi.location.lng
              console.log(obj, 'obj')
              that.$emit('searchAddress', obj)
            } else {
              that.$emit('searchAddress', '')
            }
          })
          that.map.setFitView()
        })
        // 经纬度
        that.map.on('click', function(e) {
          for (let i = 0, l = that.polygons.length; i < l; i++) {
            that.polygons[i].setMap(null)
          }
          that.marker.setMap(that.map)
          that.marker.setPosition(e.lnglat)
          geocoder.getAddress(e.lnglat, function(status, result) {
            if (status === 'complete') {
              const obj = {}
              that.province = result.regeocode.addressComponent.province
              that.city = result.regeocode.addressComponent.city
              that.district = result.regeocode.addressComponent.district
              obj.formattedAddress = result.regeocode.formattedAddress
              obj.areacode = result.regeocode.addressComponent.adcode
              that.address = result.regeocode.formattedAddress
              obj.lat = e.lnglat.lat
              obj.lng = e.lnglat.lng
              // console.log(result, 'obj')
              that.$emit('pickedAddress', obj)
            } else {
              that.$emit('pickedAddress', '')
            }
          })
        })
        // 根据坐标点定位,获取地址的详细信息
        if (that.position) {
          that.marker.setMap(that.map)
          that.marker.setPosition(that.position)
          geocoder.getAddress(that.position, function(status, result) {
            if (status === 'complete') {
              const obj = {}
              obj.formattedAddress = result.regeocode.formattedAddress
              obj.areacode = result.regeocode.addressComponent.adcode
              that.address = result.regeocode.formattedAddress
              that.$emit('positionAddress', obj)
              that.province = result.regeocode.addressComponent.province
              that.city = result.regeocode.addressComponent.city
              that.district = result.regeocode.addressComponent.district
            }
          })
          that.map.setFitView()
        } else {
          return false
        }
      })
    },
    // 根据坐标加载地图
    loadmap(val) {
      // console.log(val, 'val')
      const params = val
      const that = this
      // 插件
      for (let j = 0, k = that.polygons.length; j < k; j++) {
        that.polygons[j].setMap(null)
      }
      AMap.plugin(['AMap.DistrictSearch', 'AMap.Polygon'], function() {
        // 标注
        const opts = {
          subdistrict: 1,
          extensions: 'all',
          showbiz: false
        }
        const district = new AMap.DistrictSearch(opts)// 注意：需要使用插件同步下发功能才能这样直接使用
        function getData(data) {
          // 清除地图上的所有覆盖物
          // console.log(data)
          that.areacode = data.citycode
          that.adcode = data.adcode
          // console.log(that.adcode)
          const bounds = data.boundaries
          if (data.level === 'country') {
            that.provinces = data.districtList
            that.citys = []
            that.districts = []
          } else if (data.level === 'province') {
            that.map.remove(that.marker)
            that.citys = data.districtList
            that.districts = []
            that.address = ''
          } else if (data.level === 'city') {
            that.districts = data.districtList
          }
          if (data.level === 'country' && bounds) {
            return false
          } else {
            for (let i = 0, l = bounds.length; i < l; i++) {
              const polygon = new AMap.Polygon({
                map: that.map,
                strokeWeight: 1,
                strokeColor: '#CC66CC',
                fillColor: '#CCF3FF',
                fillOpacity: 0.5,
                path: bounds[i]
              })
              that.polygons.push(polygon)
            }
            that.map.setFitView()
          }
        }
        const sear = val ? params : '中国'
        district.search(sear, function(status, result) {
          if (status === 'complete') {
            getData(result.districtList[0])
          }
        })
      })
    },
    selectP(val) {
      this.loadmap(val)
      this.city = ''
      this.district = ''
    },
    selectC(val) {
      this.loadmap(val)
      this.district = ''
    },
    selectD(val) {
      this.loadmap(val)
    }
  }
}
</script>
<style lang="scss" scoped>
#mapContainer {
  width: 80%;
  height: 400px;
}
.c-amap_select {
  width: 100px;
  height: 30px;
  border-radius:4px;
}
.c-amap_select:not(:first-child){
  margin: 0 5px;
}
</style>

```

## 五、效果

​	![](https://raw.githubusercontent.com/Pizhong/PicGoImg/main/image-20211103150358276.png)
