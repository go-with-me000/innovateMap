<template>
  <div style="margin-top:240px;">
    <div style="width:46%;position:relative;left:27%;top:20px;">
      <div style="display:flex">
        <v-distpicker
          @selected="sel"
          :province="select.province"
          :city="select.city"
          :area="select.area"
        ></v-distpicker>

        <Input
          v-model="select.address"
          placeholder="请输入详细地址"
          size="large"
          style="width: 210px;margin-left:5px;"
        />
        <Button
          size="large"
          type="primary"
          ghost
          style="margin-left:5px;font-size:20px"
          @click="codeAddress()"
        >搜索</Button>
      </div>
    </div>
    <div id="container"></div>
  </div>
</template>

<script>
import VDistpicker from "v-distpicker";
import axios from "axios";
import qs from "qs";
export default {
  data() {
    return {
      data: [],
      addressZone: "",
      map: "",
      city: "",
      geocoder: "",
      select: {
        province: "",
        city: "",
        area: "",
        address: ""
      },

      location: {
        latitude: "",
        longitude: ""
      },
      address: {
        lng: "",
        lat: ""
      },
      score: {
        normal: 0,
        train: 0
      },
      citydata: "",
      province: "",
      markersArray: [],
      groundcity: "",
      first: true,
      latLng: "",
      map: "",
      check: false,
      transit: "",
      check2: false,
      longZone: 0,
      citys: []
    };
  },
  components: { VDistpicker },
  methods: {
    sel: function(data) {
      this.citydata = data.province.value + data.city.value + data.area.value;
      this.province = data.province.value;
    },
    init() {
      var that = this;
      //调用地址解析类
      this.geocoder = new qq.maps.Geocoder({
        complete: function(result) {
          that.address.lat = result.detail.location.lat;
          that.address.lng = result.detail.location.lng;
        }
      });
      this.map = new BMap.Map("container");
      this.map.centerAndZoom(new BMap.Point(121.393402, 31.31601), 8);
      this.transit = new BMap.TransitRoute(this.map, {
        renderOptions: {
          map: this.map,
          autoViewport: true
        },
        onSearchComplete: function(results) {
          var lineNum = results.getPlan(0).getNumLines();
          that.longZone = lineNum;

          for (var i = 0; i < lineNum; i++) {
            var line = results.getPlan(0).getLine(i);
            if (line.type == 3) {
              console.log(line);

              for (var j = 0; j < line.Tq.length; j++) {
                var latLng = new qq.maps.LatLng(line.Tq[j].lat, line.Tq[j].lng);
                that.geocoder = new qq.maps.Geocoder({
                  complete: function(result) {
                    if (
                      result.detail.addressComponents.city ==
                      result.detail.addressComponents.province
                    ) {
                      that.citys.push(result.detail.addressComponents.district);
                    } else {
                      that.citys.push(result.detail.addressComponents.city);
                    }
                  }
                });
                that.geocoder.getAddress(latLng);
              }
            }
          }
          setTimeout(function() {
            that.loadData();

            // console.log(that.citys)
          }, 1000);
        },
        intercityPolicy: BMAP_INTERCITY_POLICY_EARLY_START,
        // 配置跨城公交的交通方式策略为飞机优先
        transitTypePolicy: BMAP_TRANSIT_TYPE_POLICY_TRAIN,
        transitPolicy: BMAP_TRANSIT_POLICY_LEAST_WALKING
      });
    },

    sleep(numberMillis) {
      var now = new Date();
      var exitTime = now.getTime() + numberMillis;
      while (true) {
        now = new Date();
        if (now.getTime() > exitTime) return;
      }
    },
    loadData() {
      let URL = `http://epidemic.songcm.cn/api/epidemic/areaCal`;
      
      // this.citys=JSON.stringify(this.citys);
     
      // // this.citys=qs.stringify(this.citys);
      // this.citys = this.citys.split(",");
      
     axios({
        url: URL,
        method: "post",
        data: {
          // 'areaList': ["石家庄市","石家庄市","郑州市","郑州市","郑州市","郑州市","商丘市","商丘市","徐州市","宿州市","南京市","镇江市","无锡市","苏州市","闵行区","闵行区","宿州市"],
          'areaList':this.citys
        },
        headers: {
          "Content-Type": "application/json" //必须设置传输方式
        },
        
      })
        .then(res => {
          console.log(res);
          if (res.data.code == 200) {
            this.score.train = res.data.data.number;
            console.log(this.score.train)
          } else {
            this.$Message.warning(`${res.data.message}`);
          }
        })
        .catch(err => {
          this.$Message.warning("服务器出错");
        });
    },
    codeAddress() {
      this.check2 = true;
      this.addressZone = this.citydata + this.select.address;
      this.geocoder.getLocation(this.addressZone);
      var that = this;
      setTimeout(function() {
        var start = new BMap.Point(that.address.lng, that.address.lat);
        var end = new BMap.Point(121.393402, 31.31601);
        that.transit.search(start, end);
      }, 2000);
    }
  },
  mounted() {
    // this.addressZone =
    //   this.select.province + this.select.city + this.select.area;
    this.init();
  },
  watch: {}
};
</script>

<style scoped>
body {
  height: 50vh;
}
body,
button,
input,
select,
textarea {
  font: 12px/16px Verdana, Helvetica, Arial, sans-serif;
}
#info {
  width: 603px;
  padding-top: 3px;
  overflow: hidden;
}
.btn {
  width: 112px;
}
#container {
  position: relative;
  left: 27%;
  top: 40px;
  min-width: 600px;
  /* min-height:767px; */
  width: 46%;
  height: 80vh;
  margin-bottom: 60px;
}
ul li h3 {
  font-size: 24px;
  font-weight: bold;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  margin-bottom: 10px;
}
ul li #district {
  font-size: 24px;
  margin-bottom: 10px;
}
ul li p {
  font-size: 16px;
  font-weight: bold;
  /* font-family: Verdana, Geneva, Tahoma, sans-serif; */
  margin-bottom: 10px;
}
#location {
  position: relative;
  left: 27%;
  top: 30px;
  /* padding-top: 80px; */
  min-width: 600px;
  /* min-height:767px; */
  width: 46%;
}
.search {
  position: relative;
  left: 15%;
  width: 70%;
  top: 20px;
}
.medium {
  margin-top: 15%;
  position: relative;
  right: 10px;
}
</style>
<style>
.ivu-tag-size-medium {
  height: 50px;
  width: 150px;
  line-height: 48px;
}
</style>