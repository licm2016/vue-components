<template>
  <div :class="classes">
    <div :class="[cssPrefix + 'search-inner',this.fixed ? cssPrefix+'search-fixed flexbox' : '']">
      <form @submit="submitHandler">
        <flexbox :class="[cssPrefix + 'search']">
          <button :class="[cssPrefix + 'search-cancel']" type="button" @click="cancelHandler" v-if="fixed">
            <icon>&#xe660;</icon>
          </button>
          <x-input
            class="flexbox-item"
            htmlType="search"
            :placeholder="placeholder"
            :readonly="readonly"
            :value="value"
            :disabled="disabled" 
            :autocomplete="autocomplete"
            :autofocus="autofocus"
            :maxlength="maxlength"
            :name="name" 
            :clear="clear"
            :required="required"
            @on-focus="focusHandler"
            @on-blur="blurHandler"
            @on-keyup="keyupHandler"
            @on-keydown="keydownHandler" 
            @input="inputHandler"
            @on-change="changeHandler"
          >
            <icon slot="icon" :class="[cssPrefix + 'search-icon']">&#xe651;</icon>
          </x-input>
        </flexbox>
      </form>
      <flexbox-item :class="[cssPrefix + 'search-container']" v-if="fixed">
        <div v-if="!value" class="keywords" @click="keywordChangeHandler">
          <slot name="keywords"></slot>
        </div>
        <div v-if="value">
          <slot name="result"></slot>
        </div>
      </flexbox-item>
    </div>
  </div>
</template>

<script>
import { cssPrefix } from 'utils/variable.js'
import { input, historyPush } from 'utils/mixins.js'
import XInput from '../input'
import Icon from '../icon'
import {Flexbox, FlexboxItem} from '../flexbox'
export default {
  mixins: [input, historyPush],
  components: {
    XInput,
    Flexbox,
    FlexboxItem,
    Icon
  },
  props: {
    history: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    classes () {
      return [cssPrefix + 'search-wrapper']
    }
  },
  watch: {
    isFocus (value) {
      value && (this.fixed = true)
    },
    fixed (value) {
      if (value) {
        this.pushState()
      } else {
        this.goBack()
      }
    }
  },
  destroyed () {
    this.childFixed && this.childFixed.parentNode.removeChild(this.childFixed)
  },
  data () {
    return {
      cssPrefix: cssPrefix,
      fixed: false
    }
  },
  methods: {
    focusHandler (e) {
      this.isFocus = this.fixed = true
      if (this.$el.children[0]) {
        this.childFixed = this.$el.children[0]
        this.childFixed && document.body.appendChild(this.childFixed)
        this.childFixed.querySelector('input').focus()
      }
      this.$emit('on-focus', e)
    },
    inputHandler (value) {
      this.$emit('input', value)
    },
    changeHandler (value) {
      this.$emit('input', value)
    },
    cancelHandler () {
      this.fixed = false
      this.childFixed && this.$el.appendChild(this.childFixed)
    },
    keywordChangeHandler (e) {
      let target = e.target.classList.contains('keyword') ? e.target : e.target.closest && e.target.closest('keyword') ? e.target.closest('keyword') : null
      if (target) {
        let value = target.dataset.value || target.innerText
        this.$emit('input', value).$emit('on-submit', value)
      }
    },
    submitHandler (e) {
      e.stopPropagation()
      e.preventDefault()
      this.value && this.$emit('on-submit', this.value)
    },
    popStateBack () {
      this.cancelHandler()
    }
  }
}
</script>

<style lang="scss">
  @import '~styles/variable.scss';
  @import '~styles/mixins.scss';
  .#{$css-prefix}{
    &search{
      position:relative;
      background:#ddd;
      padding:1px;
      .#{$css-prefix}input-wrapper{
        height:0.7rem;
        margin: 4px;
        border-radius:1rem;
        &:before,&:after{
          display:none;
        }
      }
      .#{$css-prefix}input-clear-button{
        width:40px;
        .#{$css-prefix}iconfont{
          font-size:14px;
        }
      }
      input{
        padding-left:30px;
        -webkit-appearance: none;
        appearance: none;
      }
      input::-webkit-search-cancel-button{
        display:none;
      }
      input::search-cancel-button{
        display:none;
      }
      &-cancel{
        @include button;
        .#{$css-prefix}iconfont{
          font-size:20px;
          position:relative;
          top:2px;
        }
        color:#fff;
      }
    }
    &search-fixed{
      position: fixed;
      top: 0;
      left: 0;
      height: 100%;
      background: #fff;
      width: 100%;
      flex-direction: column;
      z-index: 100;
      .#{$css-prefix}search{
        background:$primary-color;
      }
    }
    &search-container{
      .keywords{
        padding:15px;
        .keyword{
          display:inline-block;
          border:1px solid #eee;
          border-radius:30px;
          vertical-align: middle;
          padding:6px 15px;
          margin:5px;
          @include active;
        }
      }
    }
  }
</style>
