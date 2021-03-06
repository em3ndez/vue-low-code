<template>
  <div :class="['qux-combo', cssClass,{ 'qux-error': hasError }, { 'qux-open': isOpen }]">
    <input class="qux-combo-input" @keyup="onKeyPress" v-model="inputValue" :placeholder="placeholder" @change="onInputChange" @blur="onBlur" ref="comboInput"/>
    <div class="qux-combo-popup" v-if="isOpen && matches.length > 0">
      <span
        v-for="(o, i) in matches"
        :key="o.label"
        :class="['qux-combo-item', {'qux-combo-item-selected': i === selectedIndex}]"
        @mousedown.stop="select(o)"
      >
        {{ o.label }}
      </span>
    </div>
  </div>
</template>
<style lang="scss">
@import "../scss/qux-combo.scss";
</style>
<script>
import _Base from "./_Base.vue"
import _DND from "./_DND.vue"
import Logger from "../core/Logger"

export default {
  name: "qCombo",
  mixins: [_Base, _DND],
  data: function() {
    return {
      inputValue: "",
      isOpen: false,
      selected: null,
      matches: [],
      selectedIndex: -1
    }
  },
  computed: {
    placeholder () {
      if (this.element && this.element.props && this.element.props.placeholder) {
        return this.element.props.label
      }
      return ''
    },
    hints() {
      return this.options.map(o => {
        if (o.toLowerCase) {
          return {
            label: o,
            value: o
          }
        }
        return o
      })

    },
    selectOption() {
      if (this.element) {
        let input = this.dataBindingInput
        if (input) {
          return input
        } else {
          return this.options[0]
        }
      }
      return this.selected
    }
  },
  methods: {
    onKeyPress(e) {
      if (this.inputValue.length > 2) {
        let search = this.inputValue.toLowerCase()
        this.matches = this.hints.filter(hint => {
          return hint.label.toLowerCase().indexOf(search) >= 0
        })
        if (this.matches.length > 0) {
          this.open()
          this.handleArrows(e)
        }
      } else {
        this.close()
      }
    },
    handleArrows(e) {
      var key = e.which || e.keyCode

      if (40 == key) {
        this.selectedIndex = Math.min(this.matches.length, this.selectedIndex + 1)
        return
      }

      if (38 == key) {
        this.selectedIndex = Math.max(-1, this.selectedIndex - 1)
        return
      }

      if (27 == key) {
        this.onInputChange()
        this.close()
        return
      }

      if (13 == key) {
        Logger.log(-5, "qCombo.handleArrows()", this.selectedIndex, this.matches)
        if (this.selectedIndex >= 0 && this.selectedIndex < this.matches.length) {
          this.select(this.matches[this.selectedIndex])
          return
        }
        this.onInputChange()
        this.close()
        return
      }
      this.selectedIndex = -1
    },
    blur () {
      if (this.$refs.comboInput) {
        this.$refs.comboInput.blur()
      }
    },
    onBlur () {
      this.onInputChange()
      this.close()
    },
    open() {
      Logger.log(5, "qCombo.open()")
      this.isOpen = true
      if (this.isOpen) {
        this._bodyListener = this.on(this.body(), "mousedown", this.close)
      }
    },
    close() {
      Logger.log(-5, "qCombo.close()")
      this.isOpen = false
      this.selectedIndex = -1
      if (this._bodyListener) {
        this._bodyListener.remove()
      }
    },
    onInputChange () {
      Logger.log(-5, "qCombo.onInputChange()", this.inputValue)
      if (this.element) {
        this.onValueChange(this.inputValue, "default")
      } else {
        this.selected = this.inputValue
        this.$emit("change", this.selected)
        this.$emit("input", this.selected)
      }
    },
    select(option) {
      Logger.log(-5, "qCombo.select()", option.value)
      if (this.element) {
        this.onValueChange(option.value, "default")
        Logger.log( 5,"qCombo.toggle() >" + this.dataBindingInputPath, option.value)
      } else {
        this.selected = option.value
        this.$emit("change", this.selected)
        this.$emit("input", this.selected)
        Logger.log(5, "qCombo.select() >" + this.selected)
      }
      this.inputValue = option.label
      this.close()
    }
  },
  watch: {},
  destroyed() {
    this.close()
  },
  mounted() {
    Logger.log(1, "qCombo.mounted() enter", this.value)
    if (!this.element) {
      this.inputValue = this.value
    }
  }
}
</script>
