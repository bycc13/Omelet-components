<template>
  <div :class="[
      type === 'textarea' ? 'om-textarea' : 'om-input',
      size ? 'om-input--' + size : '',
      {
        'is-disabled': disabled,
        'om-input-group': $slots.prepend || $slots.append,
        'om-input-group--append': $slots.append,
        'om-input-group--prepend': $slots.prepend
      }
    ]">
    <template v-if="type !== 'textarea'">
      <div class="om-input-group__prepend" v-if="$slots.prepend">
        <slot name="prepend"></slot>
      </div>
      <slot name="icon">
        <i class="om-input__icon om-icon" :class="[
              'om-icon-' + icon,
              onIconClick ? 'is-clickable' : ''
            ]" v-if="icon" @click="handleIconClick">
        </i>
      </slot>
      <input v-if="type !== 'textarea'" class="om-input__inner" v-bind="$props" :autocomplete="autoComplete" :value="currentValue" ref="input" @input="handleInput" @focus="handleFocus" @blur="handleBlur">
      <i class="om-input__icon om-icon om-icon-loading" v-if="validating"></i>
      <div class="om-input-group__append" v-if="$slots.append">
        <slot name="append"></slot>
      </div>
    </template>
    <textarea v-else class="om-textarea__inner" :value="currentValue" @input="handleInput" ref="textarea" v-bind="$props" :style="textareaStyle" @focus="handleFocus" @blur="handleBlur">
    </textarea>
  </div>
</template>
<script>
import emitter from '../mixins/emitter';
import calcTextareaHeight from './calcTextareaHeight';
import merge from '../utils/merge';

export default {
  name: 'OmInput',

  componentName: 'OmInput',

  mixins: [emitter],

  data() {
    return {
      currentValue: this.value,
      textareaCalcStyle: {}
    };
  },

  props: {
    value: [String, Number],
    placeholder: String,
    size: String,
    resize: String,
    readonly: Boolean,
    autofocus: Boolean,
    icon: String,
    disabled: Boolean,
    type: {
      type: String,
      default: 'text'
    },
    name: String,
    autosize: {
      type: [Boolean, Object],
      default: false
    },
    rows: {
      type: Number,
      default: 2
    },
    autoComplete: {
      type: String,
      default: 'off'
    },
    form: String,
    maxlength: Number,
    minlength: Number,
    max: {},
    min: {},
    step: {},
    validateEvent: {
      type: Boolean,
      default: true
    },
    onIconClick: Function
  },

  computed: {
    validating() {
      return this.$parent.validateState === 'validating';
    },
    textareaStyle() {
      return merge({}, this.textareaCalcStyle, { resize: this.resize });
    }
  },

  watch: {
    'value'(val, oldValue) {
      this.setCurrentValue(val);
    }
  },

  methods: {
    handleChange(event) {
      this.$emit('change', this.currentValue);
    },
    handleBlur(event) {
      this.$emit('blur', event);
      this.$emit('changed', this.currentValue)
      if (this.validateEvent) {
        this.dispatch('OmFormItem', 'mp.form.blur', [this.currentValue]);
      }
    },
    inputSelect() {
      this.$refs.input.select();
    },
    resizeTextarea() {
      if (this.$isServer) return;
      var { autosize, type } = this;
      if (!autosize || type !== 'textarea') return;
      const minRows = autosize.minRows;
      const maxRows = autosize.maxRows;

      this.textareaCalcStyle = calcTextareaHeight(this.$refs.textarea, minRows, maxRows);
    },
    handleFocus(event) {
      this.$emit('focus', event);
    },
    handleInput(event) {
      const value = event.target.value;
      this.$emit('input', value);
      this.setCurrentValue(value);
      this.$emit('change', value);
    },
    handleIconClick(event) {
      if (this.onIconClick) {
        this.onIconClick(event);
      }
      this.$emit('click', event);
    },
    setCurrentValue(value) {
      if (value === this.currentValue) return;
      this.$nextTick(_ => {
        this.resizeTextarea();
      });
      this.currentValue = value;
      if (this.validateEvent) {
        this.dispatch('OmFormItem', 'mp.form.change', [value]);
      }
    }
  },

  created() {
    this.$on('inputSelect', this.inputSelect);
  },

  mounted() {
    this.resizeTextarea();
  }
};
</script>