<template>
  <input @click="click" @mousedown="startInUse" @mouseup="endInUse" @touchstart="startInUse" @touchend="endInUse" @touchcancel="endInUse" :class="classObject" type="range" :min="min" v-model="realValue" :max="max" :step="step" :name="name" :orient="vertical && 'vertical'" :disabled="disabled" number>
</template>

<script>
export default {
  props: {
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 1
    },
    step: {
      type: Number,
      default: 0.01
    },
    value: {
      type: Number,
      default: 0
    },
    type: String,
    name: String,
    size: String,
    isFullwidth: Boolean,
    disabled: Boolean,
    // orientation:
    vertical: Boolean
  },
  data () {
    return {
      realValue: this.value,
      inUse: false
    }
  },
  beforeMount () {
    if (this.max < this.min) {
      throw 'Unexpected range setting: Maximum cannot be less than minimum'
    }
    this.update(this.value)
  },
  mounted () {
    this.$el.style.setProperty('--low', this.low)
    this.$el.style.setProperty('--high', this.high)
  },
  watch: {
    realValue (newVal, oldVal) {
      if (Number(newVal) !== Number(oldVal)) {
        this.$el.style.setProperty('--high', this.high)
        this.$emit('change', newVal)
      }
    },
    value (val) {
      this.update(val)
    }
  },
  methods: {
    click(e) {
      // Hack for cordova, jump to the location
      if (CORDOVA_BUILD) {
        var newValue = (e.target.max / e.target.offsetWidth) * e.offsetX
        newValue = Math.round(newValue / 10) * 10
        newValue = newValue > this.max ? this.max : newValue
        newValue = newValue < this.min ? this.min : newValue
        this.realValue = newValue
      }
    },
    startInUse (e) {
      this.inUse = true
    },
    endInUse (e) {
      this.inUse = false
    },
    update (val) {
      if (this.inUse) {
        return
      }
      if (val > this.max) {
        this.realValue = this.max
      } else if (val < this.min) {
        this.realValue = this.min
      } else {
        this.realValue = val
      }
    }
  },

  computed: {
    low () {
      return '0%'
    },
    high () {
      return (this.realValue - this.min) / (this.max - this.min) * 100 + '%'
    },
    classObject () {
      const { type, size, isFullwidth } = this
      return {
        slider: true,
        [`is-${type}`]: type,
        [`is-${size}`]: size,
        'is-fullwidth': isFullwidth
      }
    }
  }
}
</script>
