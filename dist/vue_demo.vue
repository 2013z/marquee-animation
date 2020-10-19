<template>
  <div class="marquee">
    <div class="marquee-wrap">
      <el-tabs
        v-model="activeTabName"
        :stretch="true"
        @tab-click="moveBufferList"
      >
        <el-tab-pane
          v-for="(item, key) in options"
          :key="key"
          :disabled="isRunning"
          :label="item.name"
          :name="item.name"
        ></el-tab-pane>
      </el-tabs>
      <input type="hidden" />
      <div
        class="lists-wrap"
        @mouseout="handleMouseMove"
        @mouseover="handleMouseMove"
      >
        <div :class="{ 'animate-list': true, 'is-hidden': isRunning }">
          <div class="list">
            <div
              v-for="(item, key) in getList.concat(getList)"
              :key="key"
              class="block-a-b-c"
              :data-index="key % getList.length"
            >
              <div class="content">{{ item.title }}</div>
            </div>
          </div>
        </div>
        <div :class="{ 'buffer-list': true, 'update-index': isRunning }">
          <div class="list move-buffer-list-target">
            <div v-for="(item, key) in getBufferList" :key="key" class="block">
              <div class="content">{{ item.title }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { Vue, Component, Prop } from "nuxt-property-decorator";
import anime from "./js/anime.js";

// mark: 存在几个问题：
//      1.box-shadow 过边界出会被overflow 阴影需要用inset
@Component({})
export default class Marquee extends Vue {
  @Prop({ default: () => ({}) }) options: any;
  @Prop({ default: 0 }) gap: any;
  @Prop({ default: 0 }) width: any;
  @Prop({ default: 0 }) height: any;

  animate: any;
  isPause = false;
  animationProp = {};
  isRunning = false;
  activeTabName = "";
  listWidth = 0;

  get getList() {
    if (Array.isArray(this.options)) {
      let childrens: any = [];
      this.options.forEach((item: any) => {
        item.children.forEach((child: any) => {
          child.type = item.name;
          child.img = item.img;
        });
        childrens = childrens.concat(item.children);
      });
      this.listWidth = childrens.length * 333;
      childrens = childrens.concat(childrens);
      childrens.forEach((item: any, index: any) => {
        item.offsetX = index * 333;
      });
      return childrens;
    } else {
      return this.options;
    }
  }

  get getBufferList() {
    return this.getList.concat(this.getList, this.getList);
  }

  get getCustomBlockStyle() {
    if (!this.width || !this.height) {
      return {};
    } else {
      return { height: this.height, width: this.width };
    }
  }
  created() {
    this.activeTabName = this.options[0].name || "";
  }
  mounted() {
    this.initAnimate();
  }
  initAnimate() {
    const _this = this;
    const translateXList: any[] = [];
    anime.set(".block-a-b-c", {
      translateX: function (el: any, index: any) {
        translateXList[index] = {
          x: index * 333,
        };
        return index * 333;
      },
    });
    _this.animate = anime({
      targets: translateXList,
      duration: 10000,
      easing: "linear",
      x: `-=${_this.listWidth}`,
      loop: true,
      update: function (ani: any) {
        ani.set(".block-a-b-c", {
          translateX: function (el: any, index: any) {
            return translateXList[index].x % _this.listWidth;
          },
        });
      },
      change: function (a: any) {
        // 测试每一个滑块所需要的时间差 -> 会有10毫秒左右的误差 几乎不影响，从这里获取到到[0]-[1]差值用于seek到倍数参
        if (
          a.animations[0].currentValue >= -5 &&
          a.animations[0].currentValue < 5
        ) {
          console.log("[0]", a.currentTime);
        }
        if (
          a.animations[1].currentValue >= -5 &&
          a.animations[1].currentValue < 5
        ) {
          console.log("[1]", a.currentTime);
        }
        // 测试联动效果
        if (
          a.animations[1].currentValue > -660 &&
          a.animations[1].currentValue < 0
        ) {
          _this.activeTabName = "教育";
        }
      },
    });
    // @ts-ignore 调试
    window.animate = _this.animate;
  }
  getCssRulesStyle(name: any) {
    let rules = [];
    for (let i = 0; i < document.styleSheets.length; i++) {
      // @ts-ignore
      if (document.styleSheets[i].cssRules) {
        // @ts-ignore
        rules = document.styleSheets[i].cssRules || [];
      } else {
        // @ts-ignore
        rules = document.styleSheets[i].rules || [];
      }
      for (let j = 0; j < rules.length; j++) {
        if (rules[j].selectorText === name) {
          return rules[j].style;
        }
      }
    }
    return "";
  }
  resetBufferListOffsetX() {
    this.animate.pause();
    const blockList = document.querySelectorAll(".block-a-b-c");
    let firstDom;
    let firstIndex = 1;
    let translateX = 0;
    // 获取到窗口内，左边第一个block
    for (let i = 0; i < blockList.length; i++) {
      const style = blockList[i].getAttribute("style");
      // @ts-ignore
      translateX = style.match(/\d+\.\d+/) * 1 || 0;
      if (translateX <= 333 && translateX > 0) {
        firstDom = blockList[i];
        // @ts-ignore
        firstIndex = firstDom.dataset.index * 1 + 1;
        break;
      }
    }
    const bufferList = document.querySelector(".buffer-list");
    if (bufferList) {
      const moveCssRulesStyle = this.getCssRulesStyle(
        ".move-buffer-list-target"
      );
      if (moveCssRulesStyle) {
        moveCssRulesStyle.transform = `translateX(${
          -this.listWidth + (firstIndex - 1) * -333 + -translateX
        }px)`;
      }
      return moveCssRulesStyle;
    }
  }
  moveBufferList() {
    if (this.isRunning) {
      return;
    }
    this.isRunning = true;
    const moveCssRulesStyle = this.resetBufferListOffsetX();
    const bufferChildrenList = document.querySelector(
      ".move-buffer-list-target"
    );
    setTimeout(() => {
      let targetIndex = 0;
      const time = 3;
      for (let i = 0; i < this.getList.length; i++) {
        if (this.getList[i].type === this.activeTabName) {
          targetIndex = i;
          break;
        }
      }
      console.log("time", time);
      if (bufferChildrenList) {
        // 如果当前是向左点击， 需要 - 1 如果是向右点击 无需 - 1
        console.log(
          "moveCssRules",
          moveCssRulesStyle.transform.match(/\d+\.+\d/g) * 1 - this.listWidth
        );
        console.log(
          "bufferChildrenList setAttribute",
          -this.listWidth + targetIndex * -333
        );

        bufferChildrenList.setAttribute(
          "style",
          `transform: translateX(${
            -this.listWidth + targetIndex * -333
          }px);transition: transform ${time}s linear;`
        );
        setTimeout(() => {
          // 屏幕移出一块所需要的时长为： 列表总长 / 块宽， 暂时不需要 存在百毫秒的偏差值。。duration的值最好为block的倍数
          // const duration = (this.animate.animatables.length * 333) / 10.0;
          // this.animate.seek(duration * (isRunRight ? targetIndex - 1 : targetIndex) || 0);

          // 利用change来测试出的ducation来使用， 不用v/s公式
          this.animate.seek(1110 * targetIndex || 0);
          this.isRunning = false;
          setTimeout(() => {
            this.animate.play();
            this.$nextTick(() => {
              bufferChildrenList.setAttribute("style", "");
              moveCssRulesStyle.transform = `transform: translateX(-${this.listWidth}px)`;
            });
          }, 280);
          // 平滑过渡
        }, time * 1000 + 300);
      }
    }, 3);
  }
  handleMouseMove(event: any) {
    console.log("event", event);
    const type = event.type;
    const match = event.target.className.match(/content/);
    if (match && type === "mouseover") {
      this.animate.pause();
    } else if (match && type === "mouseout") {
      this.animate.play();
    }
  }
}
</script>

<style>
.move-buffer-list-target {
  width: 16650px;
  height: 350px;
  position: relative;
  transform: translateX(-2310px);
}

.update-index {
  z-index: 250 !important;
  width: 100vw;
  height: 350px;
  position: relative;
  overflow: hidden;
  z-index: 223;
  top: -350px;
}
.is-hidden {
  visibility: hidden;
  opacity: 0;
}
</style>
<style scoped lang="scss">
.marquee {
  .marquee-wrap {
    height: 100%;
    display: flex;
    flex-direction: column;
    /deep/ .el-tabs {
      align-self: center;
      height: auto;
      .el-tabs__header {
        .el-tabs__nav-scroll {
          padding: 0 20px;
          &:after {
            display: none;
          }
        }
        .el-tabs__active-bar {
          width: 70px !important;
          height: 40px;
          background: #125692;
          border-radius: 2px;
          left: -16px !important;
        }
        .el-tabs__item {
          width: 70px;
          height: 40px;
          position: relative;
          z-index: 5;
          &.is-active {
            color: #fff;
          }
        }
      }
      .el-tabs__content {
        display: none;
      }
    }
    .lists-wrap {
      width: 100vw;
      height: 350px;
      overflow: hidden;
      .animate-list {
        width: 100vw;
        height: 350px;
        position: relative;
        overflow: hidden;
        z-index: 233;
        .list {
          position: relative;
          .block-a-b-c {
            width: 333px;
            height: 350px;
            position: absolute;
            line-height: 350px;
            .content {
              width: 280px;
              height: 330px;
              position: absolute;
              top: 0;
              left: 0;
              z-index: 233;
              box-shadow: 9px 10px 41px 0px rgba(57, 70, 95, 0.1);
              border-radius: 2px;
              color: red;
              background: lightblue;
            }
          }
        }
      }
      .buffer-list {
        width: 100vw;
        height: 350px;
        position: relative;
        overflow: hidden;
        z-index: 223;
        .list {
          width: 16650px;
          height: 350px;
          position: relative;
          .block {
            width: 333px;
            height: 350px;
            display: inline-block;
            line-height: 350px;
            float: left;
            .content {
              width: 280px;
              height: 330px;
              box-shadow: 9px 10px 41px 0px rgba(57, 70, 95, 0.1);
              border-radius: 2px;
              color: white;
              background: red;
            }
          }
        }
      }
    }
  }
}
@media screen and (max-width: 1000px) {
}
</style>
