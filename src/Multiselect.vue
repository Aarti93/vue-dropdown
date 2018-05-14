<template>
  <div class="multiselect">
    <div class="wrapper" ref="multiselect">
      <div class="multiselect-input" @click="handleOpenDropdown">
        <input class="search-input" v-model="searchQuery" :placeholder="placeholder" @click="handleInputClick" @input="handleInputChange" />
        <i :class="['caret', {'upward': open, 'downward': !open}]" @click="handleInputClick" />
      </div>
      <transition name="slide-fade" mode="out-in">
        <div class="multiselect-dropdown" v-if="open">
          <div v-if="searchQuery && filteredOptions.length === 0" class="no-result">{{noResultMessage}}</div>
          <div v-else>
            <div v-if="multiple">
              <div :class="['option-row','select-all', {highlight: allSelected}]" @click="handleChangeAll">
                <input type="checkbox" class="styled-checkbox" id="all" :checked="allSelected" />
                <label for="all">Select All</label>
              </div>
              <div :class="['option-row', {highlight: isChecked(option)}]" v-for="option in filteredOptions"
              :key="option[valueAttr]" @click="(e) => toggleCheckedState(e, option)">
                <input type="checkbox" :id="option[valueAttr]" class="styled-checkbox" :checked="isChecked(option)">
                <label :for="option[valueAttr]">{{option[displayAttr]}}</label>
              </div>
            </div>
            <div v-else>
              <div class="option-row" v-for="option in filteredOptions" :key="option[valueAttr]" @click="handleSelectedValue(option)">
                <span>{{option[displayAttr]}}</span>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>
    <div v-if="multiple" class="selected-vals">{{selectedOptionsString}}</div>
  </div>
</template>

<script>
export default {
  name: "multiselect",
  props: {
    valueAttr: {
      type: String,
      required: true
    },
    displayAttr: {
      type: String,
      required: true
    },
    options: {
      type: Array,
      required: true
    },
    placeholder: {
      type: String,
      default: "Select items"
    },
    noResultMessage: {
      type: String,
      default: "No results found"
    },
    multiple: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      open: false,
      searchQuery: "",
      checked: {},
      valueSelected: false // Is value selected for single select
    };
  },
  computed: {
    filteredOptions() {
      if (this.searchQuery.length > 0) {
        return this.options.filter(option => option[this.displayAttr].toLowerCase().indexOf(this.searchQuery.toLowerCase()) !== -1);
      }
      return this.options;
    },
    selectedOptionsString() {
      let selectedList = "";
      let comma = "";
      this.options.forEach((option) => {
        if (this.checked[option[this.valueAttr]]) {
          if (selectedList) {
            comma = ",";
          }
          selectedList = `${selectedList}${comma} ${option[this.displayAttr]}`;
        }
      });
      return selectedList;
    },
    allSelected() {
      return this.filteredOptions.every(
        option => this.checked[option[this.valueAttr]]
      );
    }
  },
  methods: {
    handleInputClick(e) {
      e.stopPropagation();
      this.searchQuery = "";
      this.toggleDropdown();
    },
    handleInputChange() {
      this.valueSelected = false;
    },
    toggleDropdown() {
      this.open = !this.open;
    },
    handleSelectedValue(option) {
      this.searchQuery = option[this.displayAttr];
      this.valueSelected = true;
      this.updateSelectedOptions(option);
      this.toggleDropdown();
    },
    handleOpenDropdown() {
      this.open = true;
    },
    toggleCheckedState(e, option) {
      e.stopPropagation();
      e.preventDefault();
      const checkedState = { ...this.checked };
      checkedState[option[this.valueAttr]] = !checkedState[
        option[this.valueAttr]
      ];
      this.checked = checkedState;
      const selectedOptions = this.options.filter(
        op => this.checked[op[this.valueAttr]]
      );
      this.updateSelectedOptions(selectedOptions);
    },
    updateSelectedOptions(selectedOptions) {
      this.$emit("input", selectedOptions);
    },
    isChecked(option) {
      return Boolean(this.checked[option[this.valueAttr]]);
    },
    handleChangeAll(e) {
      e.preventDefault();
      const checked = {};
      let selectedOptions = [];
      if (!this.allSelected) {
        this.filteredOptions.forEach((option) => {
          checked[option[this.valueAttr]] = true;
        });
        selectedOptions = this.options;
      }
      this.checked = checked;
      this.updateSelectedOptions(selectedOptions);
    },
    handleDocumentClick(e) {
      const el = this.$refs.multiselect;
      const target = e.target;
      if (el && el !== target && !el.contains(target)) {
        this.open = false;
        if (!this.valueSelected || this.multiple) {
          this.searchQuery = "";
        }
      }
    }
  },
  created() {
    document.addEventListener("click", this.handleDocumentClick); // To close dropdown on click outside
  },
  destroyed() {
    // important to clean up!!
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
  font-size: 12pt;
  .multiselect-dropdown {
    width: 98%;
    max-height: 250px;
    min-width: 150px;
    display: inline-block;
    overflow: scroll;
    border: 1px solid #e9ecef;
    box-shadow: 0px 0px 10px -2px grey;
    z-index: 100;
    .option-row {
      padding: 0.2rem 0.2rem 0.2rem 1rem;
      cursor: default;
      &.highlight {
        background-color: #e9ecef;
      }
      &.select-all {
        padding-top: 15px;
        padding-bottom: 5px;
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
    .search-input {
      width: 96%;
      padding: 4px 4px 4px 7px;
      border: 1px solid #e9ecef;
      box-shadow: 0px 0px 10px -2px grey;
      border-top-left-radius: 5px;
      border-bottom-left-radius: 5px;
      font-size: 0.9rem;
      min-height: 27px;
      min-width: 150px;
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
    right: 4%;
    position: absolute;
    cursor: pointer;
    &.downward {
      border-color: #6c757da6 transparent transparent;
      top: 45%;
    }
    &.upward {
      border-color: transparent transparent #6c757da6;
      bottom: 40%;
    }
  }
  .wrapper {
    display: inline-block;
    width: 35%;
  }
  .selected-vals {
    padding: 7px 4px 2px 10px;
    width: 60%;
    display: inline-block;
    padding: 4px 10px;
    border: 1px solid #e9ecef;
    box-shadow: 0px 0px 10px -2px grey;
    border-top-right-radius: 5px;
    border-bottom-right-radius: 5px;
    margin-left: -3px;
    vertical-align: top;
    min-height: 27px;
    margin-left: 1px;
    max-height: 70px;
    overflow: scroll;
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
      background: slategray
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
        content: '';
        position: absolute;
        left: 5px;
        top: 9px;
        background: white;
        width: 2px;
        height: 2px;
        box-shadow:
          2px 0 0 white,
          4px 0 0 white,
          4px -2px 0 white,
          4px -4px 0 white,
          4px -6px 0 white,
          4px -8px 0 white;
        transform: rotate(45deg);
      }
  }
}
</style>

