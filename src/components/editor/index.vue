<template>
  <div class="bin-ace-editor" :style="wrapStyles"></div>
</template>

<script>
  let ace = require('brace')
  export default {
    name: 'BAceEditor',
    props: {
      value: String,
      lang: {
        type: String,
        default: 'json'
      },
      theme: {
        type: String,
        default: 'chrome'
      },
      height: {
        type: String,
        default: '350'
      },
      width: {
        type: String,
        default: '100%'
      },
      fontSize: {
        type: Number,
        default: 12
      },
      options: Object,
      readonly: Boolean,
      wrap: {
        type: Boolean,
        default: false
      },
      snippets: {
        type: Boolean,
        default: true
      },
      styles: {
        type: Object
      }
    },
    data() {
      return {
        editor: null,
        contentBackup: ''
      }
    },
    computed: {
      wrapStyles() {
        return {
          width: this.width ? this.px(this.width) : '100%',
          height: this.height ? this.px(this.height) : '100%',
          border: '1px solid #e8e8e8',
          ...this.styles
        }
      }
    },
    methods: {
      px(n) {
        return /^\d*$/.test(n) ? `${n}px` : n
      },
      getValue() {
        return this.editor.getValue()
      },
      handleBlur(event) {
        this.$emit('on-blur', event)
        // 触发校验
        this.dispatch('BFormItem', 'on-form-blur', this.value)
      },
      dispatch(componentName, eventName, params) {
        let parent = this.$parent || this.$root
        let name = parent.$options.name

        while (parent && (!name || name !== componentName)) {
          parent = parent.$parent

          if (parent) {
            name = parent.$options.name
          }
        }
        if (parent) {
          parent.$emit.apply(parent, [eventName].concat(params))
        }
      }
    },
    watch: {
      value(val) {
        if (this.contentBackup !== val) {
          this.editor.session.setValue(val, 1)
          this.contentBackup = val
        }
      },
      theme(newTheme) {
        this.editor.setTheme('ace/theme/' + newTheme)
      },
      lang(newLang) {
        this.editor.getSession().setMode('ace/mode/' + newLang)
      },
      options(newOption) {
        this.editor.setOptions(newOption)
      },
      fontSize(newSize) {
        this.editor.setFontSize(newSize) // 设置文字大小
      },
      readonly(val) {
        this.editor.setFontSize(val) // 设置只读
      },
      height() {
        this.$nextTick(function () {
          this.editor.resize()
        })
      },
      width() {
        this.$nextTick(function () {
          this.editor.resize()
        })
      }
    },
    beforeDestroy() {
      this.editor.destroy()
      this.editor.container.remove()
    },
    mounted() {
      let vm = this
      let lang = this.lang
      let theme = this.theme

      let editor = vm.editor = ace.edit(this.$el)
      editor.getSession().setMode('ace/mode/' + lang)
      editor.setTheme('ace/theme/' + theme)
      editor.$blockScrolling = Infinity
      editor.setFontSize(this.fontSize) // 设置文字大小
      editor.setReadOnly(this.readonly) // 设置只读
      editor.getSession().setUseWrapMode(this.wrap)
      editor.setShowPrintMargin(false)
      editor.getSession().setTabSize(2)

      editor.setOptions({
        enableBasicAutocompletion: this.snippets,
        enableSnippets: this.snippets,
        enableLiveAutocompletion: this.snippets
      })

      this.$emit('on-init', editor)

      if (this.value) {
        editor.setValue(this.value, 1)
        editor.gotoLine(0, 0, false)
      }
      this.contentBackup = this.value

      editor.on('change', function () {
        let content = editor.getValue()
        vm.$emit('input', content)
        vm.$emit('on-change', content)
        vm.contentBackup = content
      })
      editor.on('blur', this.handleBlur)
      if (vm.options) {
        editor.setOptions(vm.options)
      }
    }
  }
</script>
