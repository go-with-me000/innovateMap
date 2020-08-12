<template>
  <div style="margin-top:240px;padding-bottom:80px">
    <div style="position:relative;left:27%;width:46%;top:20px">
      <h2>1.疫情确诊病例人口分布热力图</h2>
      <p
        style="margin-top:15px;color:#000"
      >该图即新冠肺炎感染人员于全国分布情况以及密集程度。由图可见，全国疫情分布多集中于东南地区，而西南地区仅有少数散发病例。总体而言，我国确诊较严重的省市有：武汉，长沙，广州等，相对较多省市有：河南，湖南，浙江等。</p>
    </div>
    <div class="heat">
      <img src="../assets/heat.jpg" style="width:100%" />
    </div>
    <div style="position:relative;left:27%;width:46%;top:50px">
      <h2>2.疫情确诊病例人口分布柱状图</h2>
      <p
        style="margin-top:15px;color:#000"
      >在这张图中，尤其是我国第一阶梯地区（如：青藏高原地区），由于海拔高，面积广，人员流动小，几乎无确诊病例。至于东南地区，以武汉为中心确诊案例向外逐渐递减，人口密集地区，农村以及交通枢纽城市（部分沿海城市）确诊人数相对较多。</p>
    </div>
    <div class="heat" style="margin-top:30px">
      <img src="../assets/grid.png" style="width:100%" />
    </div>
    <div style="position:relative;left:27%;width:46%;top:0px">
      <h2>3.疫情确诊病例人口分布柱状图</h2>
      <p
        style="margin-top:15px;color:#000"
      >此次新型冠状病毒肺炎病毒原发于武汉，本图通过大数据统计分析体现疫情从武汉市向外省市扩散情况。图中每条动态蓝线由武汉出发，指向有超过100个小区发现感染者的省市。从图中不难分析得到，此次疫情从武汉为源头，扩散地区多为周边城市，如信阳市，合肥市等；以及交通枢纽，如广州市，上海市等。其中很大一部分省市中外来务工人员比重大较大，如：蚌埠市，信阳市等。</p>
    </div>
    <!-- <h2>sad</h2> -->
    <div id="container" style="margin-bottom:60px"></div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      data: [],

      map: "",
      city: "",
      location: {
        latitude: "",
        longitude: ""
      }
    };
  },
  methods: {
    initMap() {
      var citylocation,
        marker = null;
      var that = this;
      var center = new TMap.LatLng(30.59331, 114.304692);
      //初始化地图
      var t = [
        {
          c: "#000868",
          a: "#0069E0"
        },
        {
          c: "#002FA4",
          a: "#008DE9"
        },
        {
          c: "#0037C2",
          a: "#35D4FF"
        }
      ];
      this.map = new TMap.Map("container", {
        zoom: 12, //设置地图缩放级别
        maxZoom: 6,
        minZoom: 6,
        pitch: 45, // 设置地图俯仰角
        center: center, //设置地图中心点坐标
        mapStyleId: "style2" //个性化样式
      });
      var arc = new TMap.visualization.Arc({
        pickStyle: e => {
          // 根据count分类
          let n = Math.min(Math.floor(e.count / 5e3), 2);
          return {
            color: t[n].c,
            animateColor: t[n].a
          };
        },
        animatable: true,
        opacity: 0.7, // 设置弧线透明度
        width: 4, // 设置弧线宽度
        mode: "vertical", // 弧线平面与地平面垂直
        selectOptions: {
          //拾取配置
          action: "hover", //拾取动作
          style: {
            //选中样式
            color: "#FF0000",
            animateColor: "#0000FF"
          },
          enableHighlight: true //是否使用高亮效果
        }
      }).addTo(that.map);
      arc.setData(that.data); //设置数据
    },
    loadData() {
      let URL = `http://www.pongshy.com:8081/api/getFromAndTo`;
      axios({
        url: URL,
        method: "GET",
        params: {
          count: 100,
          standard: 100
        }
      })
        .then(res => {
          if (res.data.code == 200) {
            this.data = res.data.data;

            this.initMap();
          } else {
            this.$Message.warning("数据加载出错");
          }
        })
        .catch(err => {
          this.$Message.warning("服务器出错");
        });
    }
  },
  mounted() {
    this.loadData();
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
h2 {
  color: rgb(146, 94, 94);
}
#container {
  position: relative;
  left: 27%;
  top: 70px;
  min-width: 600px;
  /* min-height:767px; */
  width: 46%;
  height: 80vh;
}
.heat {
  position: relative;
  left: 27%;
  top: 50px;
  min-width: 600px;
  /* min-height:767px; */
  width: 46%;
  height: 60vh;
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