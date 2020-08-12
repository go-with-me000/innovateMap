<template>
  <GeminiScrollbar id="main">
    <div class="nagivation">
      <img src="@/assets/top2.jpg" style="width:100%;" />
      <Row type="flex" justify="center" style="margin-top:-6px;">
        <Col span="4" style="background-color: #27548a;z-index:999"></Col>
        <Col span="20">
          <Menu :active-name="activeName" mode="horizontal" @on-select="push">
            <MenuItem name="map" @click.native="goAnchor()">
              <!-- <Icon type="md-cube" /> -->
              <span>点状图</span>
            </MenuItem>
            <MenuItem name="heat" @click.native="goAnchor()">
              <!-- <Icon type="ios-mail"></Icon> -->
              <span>热力图</span>
            </MenuItem>

            <MenuItem name="grid" @click.native="goAnchor()">
              <!-- <Icon type="ios-create-outline"></Icon> -->
              <span>蜂窝图</span>
            </MenuItem>
            <MenuItem name="analyse" @click.native="goAnchor()">
              <!-- <Icon type="md-attach" /> -->
              <span>疫情分析</span>
            </MenuItem>
            <MenuItem name="line" @click.native="goAnchor()">
              <!-- <Icon type="md-attach" /> -->
              <span>交通路线</span>
            </MenuItem>
          </Menu>
        </Col>
      </Row>
    </div>
    <div>
      <router-view></router-view>
    </div>
  </GeminiScrollbar>
</template>

<script>
import axios from "axios";
export default {
  name: "App",

  data() {
    return {
      activeName: ""
    };
  },
  methods: {
    initMenuActive() {
      this.activeName = this.$route.name;
    },
    goAnchor() {
      var element = document.getElementById("main");
      element.scrollIntoView(true);
    },
    push(name) {
      this.$router.push(name);
    }
  },
  mounted() {
    this.initMenuActive();
  }
};
</script>
<style scoped>
.my-scroll-bar {
  position: relative;
}
#main {
  background-color: rgb(233, 232, 232);
  height: 100vh;
}
.button {
  position: absolute;
  float: right;
  top: 0%;
  left: 80%;
}
.nagivation {
  position: absolute;
  width: 46%;
  left: 27%;
}

.ivu-menu-light {
  background-color: #27548a;
}

.ivu-menu-horizontal.ivu-menu-light:after {
  background-color: #27548a;
}

.ivu-menu-light.ivu-menu-horizontal .ivu-menu-item,
.ivu-menu-light.ivu-menu-horizontal .ivu-menu-submenu {
  color: #fff;
}

.ivu-menu-item {
  display: flex;
  background: #27548a;
  font-size: 16px;
  font-weight: normal;

  justify-content: center;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
  color: rgb(250, 250, 250);
  width: 110px;
}
.ivu-menu-item i {
  display: flex;
  align-items: center;
}

.ivu-menu-horizontal .ivu-menu-item,
.ivu-menu-horizontal .ivu-menu-submenu {
  padding: 0px;
}

.ivu-menu-submenu-title {
  display: flex;
  background: #27548a;
  font-size: 16px;
  font-weight: normal;
  justify-content: center;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;

  width: 110px;
}
.ivu-menu-submenu-title i {
  display: flex;
  align-items: center;
}

.ivu-menu-item,
.ivu-menu-light.ivu-menu-horizontal .ivu-menu-submenu {
  background-color: #27548a;
}

.ivu-menu-submenu-title:hover {
  color: #ebf3ff !important;
}

.ivu-select-dropdown {
  padding: 0px;
}

.ivu-menu-horizontal .ivu-menu-submenu .ivu-select-dropdown .ivu-menu-item {
  color: #fff;
  width: 120px;
  background-color: rgb(39, 82, 141);

  padding: 8px;
}

.ivu-menu-horizontal
  .ivu-menu-submenu
  .ivu-select-dropdown
  .ivu-menu-item:hover {
  color: #000;
}

.ivu-menu-item:hover {
  background-color: #e8e9ec;
}

.ivu-menu-item-selected {
  background-color: #f0faff !important;
  color: #000000 !important;

  font-size: 18px;
  font-weight: bold;
  align-content: center !important;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
}

.ivu-menu-item-selected:hover {
  background-color: #527dbd !important;
  color: #000000 !important;
  font-size: 18px;
  font-weight: bold;
  align-content: center !important;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
}

.ivu-menu-item-selected:after {
  background-color: #0044ff !important;
}
</style>
