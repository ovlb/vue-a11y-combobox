<template>
  <div
    :class="cbClasses.base"
    :aria-expanded="showsList.toString()"
    class="v-a11y-combobox"
    role="combobox"
    aria-owns="vCbResultList"
    aria-haspopup="listbox"
  >
    <label
      :class="cbClasses.label"
      class="v-a11y-combobox__label"
      for="cbInput"
      >{{ inputLabel }}</label
    >
    <input
      id="cbInput"
      ref="input"
      v-model.trim="inputValue"
      :class="cbClasses.input"
      :aria-activedescendant="getId(arrowPosition)"
      class="v-a11y-combobox__input"
      type="text"
      name="cbInput"
      role="searchbox"
      aria-autocomplete="list"
      aria-controls="vCbResultList"
      aria-multiline="false"
      @input="onChange"
      @focus="hasFocus = true"
      @keyup.down="onKeyDown"
      @keyup.up="onKeyUp"
      @keyup.enter="onEnter"
      @keyup.esc="onEscape"
    />
    <transition name="fade-up">
      <a11y-combobox-list
        v-show="showsList"
        :items="items"
        :active-item="arrowPosition"
        :aria-label="inputLabel"
        :no-results-message="noResultsMessage"
        role="listbox"
        @resultClick="onResultClick"
      />
    </transition>
  </div>
</template>

<script>
import A11yComboboxList from './A11yComboboxList.vue'

export default {
  components: {
    A11yComboboxList
  },
  props: {
    items: {
      type: Array,
      default: () => []
    },
    inputLabel: {
      type: String,
      required: true
    },
    isStyled: {
      type: Boolean,
      default: false
    },
    noResultsMessage: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      inputValue: '',
      hasFocus: false,
      arrowPosition: -1
    }
  },
  computed: {
    showsList() {
      return Boolean(this.inputValue && this.hasFocus)
    },
    cbClasses() {
      return this.isStyled
        ? {
            base: 'v-a11y-combobox--is-styled',
            label: 'v-a11y-combobox__label--is-styled',
            input: 'v-a11y-combobox__input--is-styled'
          }
        : ''
    }
  },
  mounted() {
    document.addEventListener('click', this.handleClick, false)
  },
  destroyed() {
    document.removeEventListener('click', this.handleClick, false)
  },
  methods: {
    onChange() {
      this.$emit('input', this.inputValue)
      this.hasFocus = true
      this.arrowPosition = -1
    },
    onKeyDown() {
      if (this.showsList && this.arrowPosition < this.items.length - 1) {
        this.arrowPosition++
      }
    },
    onKeyUp() {
      if (this.showsList) {
        this.arrowPosition === -1
          ? (this.arrowPosition = this.items.length - 1)
          : (this.arrowPosition = this.arrowPosition - 1)
      }
    },
    onEnter() {
      if (this.arrowPosition > -1) {
        this.$emit('foundResult', this.items[this.arrowPosition].id)
        this.inputValue = this.items[this.arrowPosition].title
        this.hasFocus = false
        this.arrowPosition = -1
      }
    },
    onEscape() {
      this.arrowPosition = -1
      this.inputValue = ''
    },
    onResultClick(id) {
      this.$emit('foundResult', this.items[id].id)
      this.hasFocus = false
      this.arrowPosition = -1
    },
    handleClick(evt) {
      if (!this.$el.contains(evt.target)) {
        this.hasFocus = false
      }
    },
    getId(id) {
      return id < 0 ? false : `vCbItem_${id}`
    }
  },
  provide() {
    return {
      isStyled: this.isStyled,
      getId: this.getId
    }
  }
}
</script>

<style lang="scss">
@import 'Styles/a11y-combobox.scss';

.fade-up-enter-active,
.fade-up-move {
  transition: all 0.3s ease-out;
}
.fade-up-move {
  transition: all 0.3s ease-out;
}
.fade-up-enter {
  opacity: 0;
  transform: translateY(1.5rem);
}
.fade-up-leave-to {
  opacity: 0;
  position: absolute;
  transform: translateY(0.5rem);
}
.fade-up-leave-active {
  transition: all 0.1333s ease-in;
}

:root {
  --v-a11y-cb-space: 0.5rem;
  --v-a11y-cb-clr-light: rgb(206, 206, 206);
  --v-a11y-cb-clr-dark: darkblue;
  --v-a11y-cb-z-index: 10;
}

.v-a11y-combobox--is-styled {
  @include v-a11y-combobox;
}

.v-a11y-combobox__label--is-styled {
  @include v-a11y-combobox__label;
}

.v-a11y-combobox__input--is-styled {
  @include v-a11y-combobox__input;
}
</style>
