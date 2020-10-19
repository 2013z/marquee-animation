<template>
  <div class="index-container">
    <marquee :options="marqueeList"></marquee>
  </div>
</template>

<script lang="ts">
import Marquee from ".vue_demo.vue";

@Component({
  components: { Marquee },
})
export default class NewIndex extends Vue {
  marqueeList = [
    {
      name: "直播",
      children: [
        { title: "直播1", iconName: "在线教育" },
        { title: "直播2", iconName: "在线教育" },
      ],
      img: "yewu_shuangshi_2x.png",
    },
    {
      name: "教育",
      children: [
        { title: "教育1", iconName: "在线教育" },
        { title: "教育2", iconName: "在线教育" },
      ],
      img: "yewu_shuangshi_2x.png",
    },
    {
      name: "社交",
      children: [
        { title: "社交1", iconName: "在线教育" },
        { title: "社交2", iconName: "在线教育" },
      ],
      img: "yewu_shuangshi_2x.png",
    },
    {
      name: "会议",
      children: [{ title: "会议1", iconName: "在线教育" }],
      img: "yewu_shuangshi_2x.png",
    },
    {
      name: "金融",
      children: [{ title: "金融1", iconName: "在线教育" }],
      img: "yewu_shuangshi_2x.png",
    },
    {
      name: "游戏",
      children: [{ title: "游戏1", iconName: "在线教育" }],
      img: "yewu_shuangshi_2x.png",
    },
  ];

  created() {
    this.initMarqueeList();
  }
  mounted() {}
  initMarqueeList() {
    this.marqueeList.forEach((item: any) => {
      item.img = require("./images/" + item.img);
    });
  }
}
</script>

<style lang="scss" scoped>
.index-container {
  .title {
    align-self: center;
    width: 568px;
    height: 46px;
    font-size: 28px;
    font-weight: 500;
    color: #333333;
    line-height: 46px;
    margin-top: 60px;
    margin-bottom: 10px;
    text-align: center;
  }
  .icon {
    align-self: center;
    width: 98px;
    height: 31px;
    background: url(../assets/img/new/bg_title_2x.png) no-repeat center;
    background-size: 100%;
    opacity: 0.8;
    margin-bottom: 40px;
  }
  .btns {
    display: flex;
    flex-wrap: nowrap;
    margin-bottom: 30px;
    align-self: center;
    .btn {
      cursor: pointer;
      width: 114px;
      height: 44px;
      display: flex;
      justify-content: center;
      align-items: center;
      border-radius: 2px;
      margin-right: 22px;
      color: #333333;
      &:last-child {
        margin-right: unset;
      }
      &.active {
        color: #f9fafc;
        background: #125692;
      }
    }
  }
  .content {
    padding-top: 50px;
    height: 500px;
    position: relative;
    .text-grid {
      margin-top: 91px;
    }
    &::before {
      display: block;
      content: "";
      position: absolute;
      width: 56px;
      height: 58px;
      background: url(../assets/img/new/bg_c1_2x.png) no-repeat center;
      background-size: 100%;
      left: 81px;
      bottom: 74px;
    }
    &::after {
      display: block;
      content: "";
      position: absolute;
      width: 122px;
      height: 283px;
      background: url(../assets/img/new/bg_c3_2x.png) no-repeat center;
      background-size: 100%;
      right: 0px;
      bottom: 94px;
    }
  }
  .swiper {
    height: 690px;
    display: flex;
    flex-direction: column;
    background: #fafbfc;
  }
}
</style>
