<template>
  <transition name="el-notification-fade">
    <div
        :class="['el-notification', customClass, horizontalClass]"
        v-show="visible"
        :style="positionStyle"
        @mouseenter="clearTimer()"
        @mouseleave="startTimer()"
        @click="click"
        role="alert"
    >
      <i
          class="el-notification__icon"
          :class="[ typeClass, iconClass ]"
          v-if="type || iconClass">
      </i>
      <div class="el-notification__group" :class="{ 'is-with-icon': typeClass || iconClass }">
        <h2 class="el-notification__title" v-text="title"></h2>
        <div class="el-notification__content" v-show="message">

          <slot>
            <p v-if="!dangerouslyUseHTMLString">{{ message }}</p>
            <p v-else v-html="message"></p>
          </slot>
        </div>
        {{timeSelectd}}
        <div style="position: absolute; top: 8px; right: 40px; width: 100px;" v-if="timeList">
          <el-select v-model="timeSelectd" placeholder="定时开启" 
          size="mini" style="float: right" @change="timeSelect">
            <el-option
                v-for="item in timeList"
                :key="item.value"
                :label="item.label"
                :value="item.value">
            </el-option>
          </el-select>

        </div>


        <div
            class="el-notification__closeBtn el-icon-close"
            v-if="showClose"
            @click.stop="close"></div>
      </div>
    </div>
  </transition>
</template>

<script type="text/babel">
import store from "store";
// import ElSelect from 'element-ui/packages/select/src/select.vue';
// import ElOption from 'element-ui/packages/select/src/option.vue';
// import ElOption2 from 'element-ui/packages/select/src/option-group.vue';
// import ElOption3 from 'element-ui/packages/select/src/select-dropdown.vue';
// // import Eldropdown from 'element-ui/packages/dropdown';
// // import ElDropdownMenu from 'element-ui/packages/dropdown-item';
// // import dropdownmenu from 'element-ui/packages/dropdown-menu';

let typeMap = {
  success: "success",
  info: "info",
  warning: "warning",
  error: "error"
};

export default {
  components: {
    // ElSelect,
    // ElOption,
    // ElOption2,
    // ElOption3
    // Eldropdown,
    // ElDropdownMenu,
    // dropdownmenu
  },
  data() {
    return {
      visible: false,
      title: "",
      message: "",
      duration: 4500,
      type: "",
      showClose: true,
      customClass: "",
      iconClass: "",
      onClose: null,
      onClick: null,
      onTimeSelect: null, //选择时间事件传出选中值
      timeSelectd: null, //当前选中的时间
      timeList: null, //传入的时间参数
      closed: false,
      verticalOffset: 0,
      timer: null,
      dangerouslyUseHTMLString: false,
      position: "top-right"
    };
  },

  computed: {
    typeClass() {
      return this.type && typeMap[this.type]
        ? `el-icon-${typeMap[this.type]}`
        : "";
    },

    horizontalClass() {
      return this.position.indexOf("right") > -1 ? "right" : "left";
    },

    verticalProperty() {
      return /^top-/.test(this.position) ? "top" : "bottom";
    },

    positionStyle() {
      return {
        [this.verticalProperty]: `${this.verticalOffset}px`
      };
    }
  },

  watch: {
    closed(newVal) {
      if (newVal) {
        this.visible = false;
        this.$el.addEventListener("transitionend", this.destroyElement);
      }
    }
  },

  methods: {
    // 时间选择器,传出一个选择的时间
    timeSelect(vals) {
      if (this.timeList.length > 0) {
        // 赋值选中时间
        this.timeSelectd = vals;
        // 把选中的间隔时间放在本地存储里
        store.set("notificationTimeSelectd", vals);
        // 设置当前选中的时间
        store.set("notificationOldTime", new Date().getTime());

        // 如果选中时间事件存在,就传出选中的值
        if (typeof this.onTimeSelect === "function") this.onTimeSelect(vals);
        this.close();
      }
    },

    destroyElement() {
      this.$el.removeEventListener("transitionend", this.destroyElement);
      this.$destroy(true);
      this.$el.parentNode.removeChild(this.$el);
    },

    click() {
      if (typeof this.onClick === "function") {
        this.onClick();
      }
    },

    close() {
      this.closed = true;
      if (typeof this.onClose === "function") {
        this.onClose();
      }
    },

    clearTimer() {
      clearTimeout(this.timer);
    },

    startTimer() {
      if (this.duration > 0) {
        this.timer = setTimeout(() => {
          if (!this.closed) {
            this.close();
          }
        }, this.duration);
      }
    },
    keydown(e) {
      if (e.keyCode === 46 || e.keyCode === 8) {
        this.clearTimer(); // detele 取消倒计时
      } else if (e.keyCode === 27) {
        // esc关闭消息
        if (!this.closed) {
          this.close();
        }
      } else {
        this.startTimer(); // 恢复倒计时
      }
    }
  },
  mounted() {
    if (this.timeList.length > 0) {
      let nowTime = new Date().getTime();
      //如果当前时间减去上次打开的时间小于选中的时间就一直关闭提示框,否则显示
      if (
        nowTime - store.get("notificationOldTime") <
        store.get("notificationTimeSelectd") * 60000
      ) {
        this.close();
      }
    }

    if (this.duration > 0) {
      this.timer = setTimeout(() => {
        if (!this.closed) {
          this.close();
        }
      }, this.duration);
    }
    document.addEventListener("keydown", this.keydown);
  },
  beforeDestroy() {
    document.removeEventListener("keydown", this.keydown);
  }
};
</script>

