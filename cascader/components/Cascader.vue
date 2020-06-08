<template>
  <van-popup :value="visible" position="bottom" @click-overlay="onPopupClose" @close="onPopupClose">
    <div class="wux-class" :class="classes.wrap">
      <div :class="classes.hd">
          <div :class="classes.title" v-if="title">{{ title }}</div>
          <div :class="classes.menus" v-if="activeOptions.length">
              <div 
                v-for="(item, index) in activeOptions" :key="index"
                :class="[classes.menu, activeIndex === index ? prefixCls + '__menu--active' : '']"
                @click="onMenuClick(index)">{{ item[fieldNames['label']] }}
              </div>
          </div>
        </div>
        <div :class="classes.bd" :style="bodyStyle">
            <div v-for="(option, optionIndex) in showOptions" :key="optionIndex" :class="classes.inner">
              <div scroll-y class="wux-scroll-view-class" :class="classes.scrollView">
                <div :class="classes.option">
                  <div
                    template v-for="(item, index) in option" :key="index"
                    :class="[classes.item, activeValue[optionIndex] === item[fieldNames['value']] ? prefixCls + '__item--active' : '', item.disabled ? prefixCls + '__item--disabled' : '']"
                    @click="onItemSelect(item, optionIndex)">
                    <span>{{ item[fieldNames['label']] }}</span>
                    <van-icon :class="classes.icon" name="success" size="16" color="#ef473a" v-if="activeValue[optionIndex] === item[fieldNames['value']]"/>
                  </div>
                </div>
              </div>
            </div>
        </div>
    </div>
  </van-popup>
</template>

<script>
import classNames from '../utils/classNames'
import arrayTreeFilter from '../utils/arrayTreeFilter'
const WUX_CASCADER = 'wux-cascader'
const defaultFieldNames = {
  label: 'label',
  value: 'value',
  children: 'children',
}
export default {
  props: {
    prefixCls: {
      type: String,
      default: 'wux-cascader',
    },
    defaultValue: {
      type: Array,
      default: () => [],
    },
    title: {
      type: String,
      default: '',
    },
    options: {
      type: Array,
      default: () => [],
    },
    chooseTitle: {
      type: String,
      default: '请选择',
    },
    visible: {
      type: Boolean,
      default: false,
    },
    defaultFieldNames: {
      type: Object,
      default: () => defaultFieldNames,
    },
  },
  data() {
    return {
      activeOptions: [],
      activeIndex: 0,
      bodyStyle: '',
      activeValue: [],
      showOptions: [],
      fieldNames: {},
      controlled: false
    }
  },
  created() {
    const { defaultValue, value, controlled } = this
    const activeValue = controlled ? value : defaultValue
    const fieldNames = Object.assign({}, defaultFieldNames, this.defaultFieldNames)

    // this.setData({ activeValue, fieldNames }, () => this.getCurrentOptions(activeValue))
    this.activeValue = activeValue
    this.fieldNames = fieldNames
    this.getCurrentOptions(activeValue)
  },
  methods: {
    getActiveOptions(activeValue) {
      const { options } = this;
      const value = this.getFieldName('value')
      const childrenKeyName = this.getFieldName('children')

      return arrayTreeFilter(options, (option, level) => option[value] === activeValue[level], { childrenKeyName })
    },
    getShowOptions(activeValue) {
      const { options } = this;
      const children = this.getFieldName('children')
      const result = this.getActiveOptions(activeValue).map((activeOption) => activeOption[children]).filter((activeOption) => !!activeOption)

      return [options, ...result]
    },
    getMenus(activeValue = [], hasChildren) {
      const { options, chooseTitle } = this
      const activeOptions = this.getActiveOptions(activeValue)
      if (hasChildren && activeOptions.length < options.length) {
        const value = this.getFieldName('value')
        const label = this.getFieldName('label')

        activeOptions.push({
          [value]: WUX_CASCADER,
          [label]: chooseTitle
        })
      }
      return activeOptions
    },
    getNextActiveValue(value, optionIndex) {
      let { activeValue } = this
      activeValue = activeValue.slice(0, optionIndex + 1)
      activeValue[optionIndex] = value
      return activeValue
    },
    updated(currentOptions, optionIndex, condition, callback) {
      
      const value = this.getFieldName('value')
      const children = this.getFieldName('children')
      const hasChildren = currentOptions[children] && currentOptions[children].length > 0
      const activeValue = this.getNextActiveValue(currentOptions[value], optionIndex)
      const activeOptions = this.getMenus(activeValue, hasChildren)
      const activeIndex = activeOptions.length - 1
      const showOptions = this.getShowOptions(activeValue)
      const params = {
        activeValue,
        activeOptions,
        activeIndex,
        showOptions,
      }
      // 判断 hasChildren 计算需要更新的数据
      if (hasChildren || (activeValue.length === showOptions.length && (optionIndex = Math.max(0, optionIndex - 1)))) {
        params.bodyStyle = {transform: `translate(${-50 * optionIndex}%)`}
        params.showOptions = showOptions
      }

      // 判断是否需要 setData 更新数据
      if (condition) {
        // this.setData(params)
        this.activeValue = params.activeValue
        this.activeOptions = params.activeOptions
        this.activeIndex = params.activeIndex
        this.showOptions = params.showOptions
        this.bodyStyle = params.bodyStyle
      }

      // 回调函数
      if (typeof callback === 'function') {
        callback.call(this, currentOptions, activeOptions, !hasChildren)
      }
    },
    /**
     * 更新级联数据
     * @param {Array} activeValue 当前选中值
     */
    getCurrentOptions(activeValue = this.activeValue) {
      const optionIndex = Math.max(0, activeValue.length - 1)
      const activeOptions = this.getActiveOptions(activeValue)
      const currentOptions = activeOptions[optionIndex]

      if (currentOptions) {
        this.updated(currentOptions, optionIndex, true)
      } else {
        const value = this.getFieldName('value')
        const label = this.getFieldName('label')

        activeOptions.push({
          [value]: WUX_CASCADER,
          [label]: this.chooseTitle
        })

        const showOptions = this.getShowOptions(activeValue)
        const activeIndex = activeOptions.length - 1
        const params = {
          showOptions,
          activeOptions,
          activeIndex,
          bodyStyle: '',
        }

      // this.setData(params)
      this.showOptions = params.showOptions
      this.activeOptions = params.activeOptions
      this.activeIndex = params.activeIndex
      this.bodyStyle = ''
      }
    },
    /**
     * 点击菜单时的回调函数
     */
    onMenuClick(menuIndex) {
      // const { menuIndex } = e.currentTarget.dataset
      const index = menuIndex > 1 ? menuIndex - 1 : 0
      const bodyStyle = {transform: `translate(${-50 * index}%)`}

      // this.setData({
      //   bodyStyle,
      //   activeIndex: menuIndex,
      // })
      this.bodyStyle = bodyStyle
      this.activeIndex = menuIndex
    },
    /**
     * 点击选项时的回调函数
     */
    onItemSelect(item, optionIndex) {
      // const { item, optionIndex } = e.currentTarget.dataset

      // 判断是否禁用
      if (!item || item.disabled) return

      // updated
      this.updated(item, optionIndex, !this.controlled, this.onChange)
    },
    /**
     * 组件关闭时的回调函数
     */
    onPopupClose() {
        // this.triggerEvent('close')
        this.$emit('close')
    },
    /**
     * 选择完成时的回调函数
     */
    onChange(currentOptions = {}, activeOptions = [], done = false) {
      const options = activeOptions.filter((n) => n[this.getFieldName('value')] !== WUX_CASCADER)
      const value = options.map((n) => n[this.getFieldName('value')])

      // 判断是否异步加载
      if (currentOptions.isLeaf === false && !currentOptions.children) {
        this.emitEvent({ value, options, done: false })
        // this.triggerEvent('load', { value, options })
        this.$emit('load', { value, options })
        return
      }

      // 正常加载
      this.emitEvent({ value, options, done })
    },
    emitEvent(params = {}) {
      // this.triggerEvent('change', params)
      this.$emit('change', params)

      // 当选择完成时关闭组件
      if (params.done) {
          this.onPopupClose()
      }
    },
    getFieldName(name) {
      return this.fieldNames[name]
    },
  },
  watch: {
    options() {
      this.getCurrentOptions()
    }
  },
  computed: {
    classes() {
      const { prefixCls } = this
      const wrap = classNames(prefixCls)
      const hd = `${prefixCls}__hd`
      const title = `${prefixCls}__title`
      const menus = `${prefixCls}__menus`
      const menu = `${prefixCls}__menu`
      const bd = `${prefixCls}__bd`
      const inner = `${prefixCls}__inner`
      const scrollView = `${prefixCls}__scroll-view`
      const option = `${prefixCls}__option`
      const item = `${prefixCls}__item`
      const icon = `${prefixCls}__icon`
      const ft = `${prefixCls}__ft`

      return {
        wrap,
        hd,
        title,
        menus,
        menu,
        bd,
        inner,
        scrollView,
        option,
        item,
        icon,
        ft,
      }
    },
  }
}
</script>

<style lang="less" scoped>
.wux-cascader__hd {
  position: relative;
  width: 100%;
  font-size: 17px;
  line-height: 1.5;
  color: #444;
  &::after {
    content: " ";
    position: absolute;
    left: 0;
    bottom: 0;
    right: 0;
    height: 1px;
    border-bottom: 1px solid #d9d9d9;
    color: #d9d9d9;
    transform-origin: 0 100%;
    transform: scaleY(.5)
  }
  .wux-cascader__title {
    position: relative;
    height: 44px;
    display: -ms-flexbox;
    display: flex;
    -ms-flex-align: center;
    align-items: center;
    -ms-flex-pack: center;
    justify-content: center;
    text-align: center;
    box-sizing: border-box;
    &:after {
      content: " ";
      position: absolute;
      left: 0;
      bottom: 0;
      right: 0;
      height: 1px;
      border-bottom: 1px solid #d9d9d9;
      color: #d9d9d9;
      transform-origin: 0 100%;
      transform: scaleY(.5)
    }
  }
  .wux-cascader__menus {
    display: -ms-flexbox;
    display: flex;
    height: 44px;
    padding: 0 10px;
    -ms-flex-align: center;
    align-items: center;
    box-sizing: border-box;
    .wux-cascader__menu {
      font-size: 13px;
      padding: 0 10px;
      max-width: 40%;
      width: auto;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      word-wrap: normal;
      &.wux-cascader__menu--active {
        color: #ef473a
      }
    }
  }
}
.wux-cascader__bd {
  width: 100%;
  display: -ms-flexbox;
  display: flex;
  transition: transform .3s;
  background-color: #f5f5f5;
  .wux-cascader__inner {
    display: block;
    height: inherit;
    width: 50%;
    -ms-flex: 0 0 50%;
    flex: 0 0 50%;
    background-color: #fff;
    &:nth-child(2n) {
      background-color: #f5f5f5
    }
  }
}
.wux-cascader__scroll-view {
  max-height: 270px;
  overflow-y: scroll;
  .wux-cascader__option {
    width: 100%;
    height: inherit;
    display: block;
    padding: 0 20px;
    box-sizing: border-box;
    .wux-cascader__item {
      position: relative;
      z-index: 10;
      display: block;
      color: #333;
      font-size: 13px;
      height: 40px;
      line-height: 40px;
      text-align: left;
      padding-right: 18px;
      width: auto;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      word-wrap: normal;
      &.wux-cascader__item--active {
        color: #ef473a
      }
      &::after {
        content: " ";
        position: absolute;
        left: 0;
        bottom: 0;
        right: 0;
        height: 1px;
        border-bottom: 1px solid #d9d9d9;
        color: #d9d9d9;
        transform-origin: 0 100%;
        transform: scaleY(.5)
      }
      .wux-cascader__icon {
        position: absolute;
        top: 12px;
        right: 0;
        z-index: 20;
        font-size: 0;
        line-height: 1;
      }
    }
  }
}
</style>