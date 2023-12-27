<!--
@Author: 左盐
@Date:   2018-03-19 12:25:54
@Email:  huabinglan@163.com
@Project: xxx
@Last modified by:   左盐
@Last modified time: 2018-04-02 12:20:29
-->
<template>
  <base-drag :options="options" :cls="cls">
    <div class="vl-notify-content" :style="contentStyle" :id="id"></div>
  </base-drag>
</template>

<script>
import helper from "./helper/helper.js";
import drag from "./components/drag/drag";

export default {
  data() {
    return {
      cls: {
        "vl-notify-iframe": true
      },
      id: "vlip" + new Date().getTime()
    };
  },
  props: {
    options: {
      type: Object,
      default: function () {
        return {};
      }
    }
  },
  components: {
    "base-drag": drag
  },
  computed: {
    contentStyle() {
      return {
        height: "calc(100% - 50px)",//parseInt(this.options.area[1]) - 50 + "px",
        minHeight: "0px",
        overflow: "auto"
      };
    }
  },
  async mounted() {
    this.getContent();
    helper.hiddenScrollBar(this.options);
  },
  methods: {
    async getContent() {
      await helper.sleep(10);
      let propsData = helper.deepClone(this.options.content.data) || {};
      propsData["layerid"] = this.options.id;
      propsData["lydata"] = this.options.content.data;
      propsData["lyoption"] = this.options;
      let instance = new this.options.content.content({
        //具体参数信息，请参考vue源码
        parent: this.options.content.parent,
        propsData: propsData
      });
      // 在数据对象中传递 `scopedSlots`
      // 格式为 { name: props => VNode | Array<VNode> }
      instance.$scopedSlots =this.options.content.slots;
      instance.vm = instance.$mount();
      document.getElementById(this.id).appendChild(instance.vm.$el);
      this.options.layer.instancesVue[this.options.id].iframe = instance.vm;
    },

    btnyes(event) {
      helper.btnyes(event, this.options);
    },
    btncancel(event) {
      helper.btncancel(event, this.options);
    },
    beforeLayerClose (layerid) {
      //关闭前通知子组件
      let allowClose = true;
      const beforeLayerCloseFun = this.options.layer.instancesVue[this.options.id].iframe.beforeLayerClose
      if(beforeLayerCloseFun){
        allowClose = beforeLayerCloseFun(layerid);
        if(allowClose == undefined){
          allowClose = true
        }
      }
      return allowClose;
    }
  }
};
</script>
