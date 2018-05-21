<template>
  <div class="multiselect">
    <div :class="['wrapper', {'multiple': multiple}]" ref="multiselect">
      <div class="multiselect-input" @click="changeDropdownState(true)">
        <input :class="['input', {'invalid': !valid, 'search': enableSearch}]" :readonly="!enableSearch"
          :placeholder="placeholder"  @keyup="handleInput" :disabled="disabled" :value="displayValue" />
        <i :class="['caret', {'upward': open, 'downward': !open, 'disabled': disabled}]" @click="handleInputClick" />
        <div class="error-msg" v-if="!valid && !open">{{invalidMsg}}</div>
      </div>
      <transition name="slide-fade" mode="out-in">
        <div class="multiselect-dropdown" v-if="open && !disabled">
          <div v-if="searchQuery && filteredOptions.length === 0" class="no-result">{{noResultMessage}}</div>
          <div v-else>
            <div v-if="multiple">
              <div :class="['option-row','select-all', {highlight: allSelected}]" @click="handleChangeAll">
                <input type="checkbox" class="styled-checkbox" id="all" :checked="allSelected" />
                <label for="all">Select All</label>
              </div>
              <div :class="['option-row', {highlight: isChecked(option)}]" v-for="option in filteredOptions"
                :key="val(option)" @click="toggleCheckedState(option, $event)">
                <input type="checkbox" :id="val(option)" class="styled-checkbox" :checked="isChecked(option)">
                <label :for="val(option)">{{display(option)}}</label>
              </div>
            </div>
            <div v-else>
              <div :class="['option-row', {highlight: val(option) === val(selectedValue)}]" v-for="option in filteredOptions"
                :key="val(option)" @click="handleSelectedValue(option)">
                <label>{{display(option)}}</label>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>
    <div v-if="multiple" :class="['selected-vals', {'disabled': disabled}]">{{selectedOptionsString}}</div>
  </div>
</template>

<script>
const SELECT_ALL = "*";
export default {
  name: "MultipleSelect",
  props: {
    value: [Array, String, Object],
    valueAttr: { type: String },
    displayAttr: { type: String },
    options: { type: Array, required: true },
    placeholder: { type: String, default: "Select items" },
    noResultMessage: { type: String, default: "No results found" },
    multiple: { type: Boolean, default: false },
    defaultSelect: { type: [String, Array, Object], default: null },
    enableSearch: { type: Boolean, default: false },
    disabled: { type: Boolean, default: false },
    valid: { type: Boolean, default: true },
    invalidMsg: { type: String, default: "This field is required" }
  },
  data() {
    const checked = {};
    let selectedValue = null;
    const selected = this.defaultSelect || this.value; // In case of default set through v-model
    if (selected) {
      if (this.multiple) {
        if (this.val(selected) === SELECT_ALL) {
          this.options.forEach((option) => {
            checked[this.val(option)] = true;
          });
        } else {
          selected.forEach((option) => {
            checked[this.val(option)] = true;
          });
        }
      } else {
        selectedValue = selected;
      }
    }
    return {
      open: false,
      searchQuery: "",
      checked,
      selectedValue,
      editSearch: false
    };
  },
  computed: {
    filteredOptions() {
      if (this.searchQuery.length > 0) {
        return this.options.filter(option => this.display(option).toLowerCase().indexOf(this.searchQuery.toLowerCase()) !== -1);
      }
      return this.options;
    },
    selectedOptionsString() {
      let selectedList = "";
      let comma = "";
      this.options.forEach((option) => {
        if (this.checked[this.val(option)]) {
          if (selectedList) {
            comma = ",";
          }
          selectedList = `${selectedList}${comma} ${this.display(option)}`;
        }
      });
      return selectedList;
    },
    allSelected() {
      return this.filteredOptions.every(option => this.checked[this.val(option)]);
    },
    selectedOptions() {
      return this.options.filter(option => this.checked[this.val(option)]);
    },
    multiSelectDisplayMsg() {
      return `${this.selectedOptions.length} of ${
        this.options.length
      } selected`;
    },
    displayValue() {
      const singleSelect = this.selectedValue || this.defaultSelect || this.value;
      if (this.enableSearch && this.editSearch) {
        return this.searchQuery;
      } else if (this.multiple) {
        return this.multiSelectDisplayMsg;
      } else if (singleSelect) {
        return this.display(singleSelect);
      }
      return "";
    }
  },
  methods: {
    val(option) {
      return (option && option[this.valueAttr]) || option;
    },
    display(option) {
      return option[this.displayAttr] || option;
    },
    updateSelectedOptions(selectedOptions) {
      this.$emit("input", selectedOptions);
    },
    handleInput(e) {
      this.searchQuery = e.target.value;
    },
    handleInputClick(e) {
      e.stopPropagation();
      this.changeDropdownState(!this.open);
    },
    changeDropdownState(state) {
      if (!this.disabled) {
        this.open = state;
        if (state) {
          if (this.enableSearch) {
            this.editSearch = true;
          }
        } else {
          this.searchQuery = "";
          this.editSearch = false;
        }
      }
    },
    handleSelectedValue(option) {
      this.selectedValue = option;
      this.changeDropdownState(false);
      this.updateSelectedOptions(option);
    },
    toggleCheckedState(option, e) {
      e.stopPropagation();
      e.preventDefault();
      const checkedState = { ...this.checked };
      checkedState[this.val(option)] = !checkedState[this.val(option)];
      this.checked = checkedState;
      this.updateSelectedOptions(this.selectedOptions);
    },
    isChecked(option) {
      return Boolean(this.checked[this.val(option)]);
    },
    handleChangeAll(e = {}) {
      e.preventDefault();
      const checked = {};
      this.filteredOptions.forEach((option) => {
        checked[this.val(option)] = !this.allSelected;
      });
      this.checked = checked;
      this.updateSelectedOptions(this.selectedOptions);
    },
    handleDocumentClick(e) {
      const el = this.$refs.multiselect;
      const target = e.target;
      if (el && el !== target && !el.contains(target)) {
        this.changeDropdownState(false);
      }
    }
  },
  created() {
    document.addEventListener("click", this.handleDocumentClick); // To close dropdown on click outside
  },
  destroyed() {
    document.removeEventListener("click", this.handleDocumentClick);
  }
};
</script>

<style lang="scss" scoped>
.slide-fade-enter-active {
  transition: all 0.4s ease;
}
.slide-fade-leave-active {
  transition: all 0.2s ease;
}
.slide-fade-enter {
  transform: translateY(-5px);
  opacity: 0;
}
.slide-fade-leave-to {
  transform: translateY(-5px);
  opacity: 0;
}

.multiselect {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  width: 100%;
  font-size: 11pt;
  text-align: left;
  .multiselect-dropdown {
    width: 100%;
    max-height: 250px;
    min-width: 150px;
    display: inline-block;
    overflow: scroll;
    border: 1px solid #e9ecef;
    box-shadow: 0px 0px 10px -2px grey;
    z-index: 100;
    position: absolute;
    background-color: white;
    font-weight: 400;
    .option-row {
      padding: 0.2rem 0.2rem 0.2rem 1rem;
      cursor: default;
      &.highlight {
        background-color: #e9ecef;
        font-weight: 500;
      }
      &.select-all {
        padding-top: 15px;
        border-bottom: 1px solid #e9ecef;
      }
    }
    .option-row:hover {
      background-color: #e9ecef;
    }
    span {
      margin-left: 0.2rem;
    }
    .no-result {
      padding: 10px;
    }
  }

  .multiselect-input {
    width: 100%;
    position: relative;
    .input {
      width: 100%;
      padding: 4px 4px 4px 7px;
      border: 1px solid #e9ecef;
      box-shadow: 0px 0px 10px -2px grey;
      border-radius: 5px;
      font-size: 0.9rem;
      min-height: 27px;
      cursor: default;
      &.invalid {
        border-color: #dc3545;
      }
      &.search {
        cursor: auto;
      }
    }
    .error-msg {
      margin-top: 0.25rem;
      font-size: 80%;
      color: #dc3545;
    }
    input.search-input:focus {
      outline: none;
    }
  }
  .caret {
    border-width: 6px;
    border-style: solid;
    border-right: 5px solid transparent;
    border-left: 5px solid transparent;
    right: 10px;
    position: absolute;
    cursor: pointer;
    &.downward {
      border-color: #6c757da6 transparent transparent;
      &.disabled {
        border-color: #54545436 transparent transparent;
      }
      top: 15px;
    }
    &.upward {
      border-color: transparent transparent #6c757da6;
      &.disabled {
        border-color: transparent transparent #54545436;
      }
      top: 8px;
    }
  }
  .wrapper {
    display: inline-block;
    width: 100%;
    position: relative;
    &.multiple {
      width: 38%;
    }
  }
  .selected-vals {
    padding: 7px 4px 2px 10px;
    width: 60%;
    padding: 4px 10px;
    border: 1px solid #e9ecef;
    box-shadow: 0px 0px 10px -2px grey;
    border-radius: 5px;
    margin-left: -3px;
    vertical-align: top;
    min-height: 30px;
    margin-left: 1px;
    max-height: 70px;
    overflow: scroll;
    float: right;
  }
  input:disabled {
    color: #54545473;
  }
  .disabled {
    color: #54545473;
  }
  .styled-checkbox {
    position: absolute; // take it out of document flow
    opacity: 0; // hide it

    & + label {
      position: relative;
      cursor: pointer;
      padding: 0;
    }

    // Box.
    & + label:before {
      content: "";
      margin-right: 10px;
      display: inline-block;
      vertical-align: text-top;
      width: 18px;
      height: 18px;
      background: white;
      border: 1px solid slategrey;
      border-radius: 7px;
    }

    // Box hover
    &:hover + label:before {
      background: slategray;
    }

    // Box focus
    &:focus + label:before {
      box-shadow: 0 0 0 3px rgba(0, 0, 0, 0.12);
    }

    // Box checked
    &:checked + label:before {
      background: slategray;
    }

    &:checked + label:after {
      content: "";
      position: absolute;
      left: 5px;
      top: 9px;
      background: white;
      width: 2px;
      height: 2px;
      box-shadow: 2px 0 0 white, 4px 0 0 white, 4px -2px 0 white,
        4px -4px 0 white, 4px -6px 0 white, 4px -8px 0 white;
      transform: rotate(45deg);
    }
  }
  label {
    margin-top: 0.2rem;
    margin-bottom: 0.2rem;
    display: inline-block;
  }
}
</style>
