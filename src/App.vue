<template>
    <div id="app">
       <VueMultipleSelect :options="options" valueAttr="age" displayAttr="name" v-model="selectedOptions" multiple :valid="false"/>
    </div>

</template>

<script>

// import Multiselect from 'vue-select-multiple';
import VueMultipleSelect from '../dist/vue-select-multiple';
// import { VueMultipleSelect } from '../dist/vue-select-multiple';
// import { VueMultipleSelect } from './components';

export default {
  name: 'app',
  data() {
    return {
      options: ["Ab", "Bjdfkf", "CBjdfkf", "DBjdfkf"],
      options1: [
        {
          name: 'Aarti',
          age: 10,
        },
        {
          name: 'Gaurav',
          age: 11,
        },
        {
          name: 'Peter',
          age: 12,
        },
        {
          name: 'Alice',
          age: 13,
        },
        {
          name: 'Bob',
          age: 14,
        },
        {
          name: 'Adlyn',
          age: 15,
        },
        {
          name: 'Sonia',
          age: 16,
        },
        {
          name: 'Rahul',
          age: 17,
        },
        {
          name: 'Rohit',
          age: 18,
        },
        {
          name: 'Mohit',
          age: 19,
        },
        {
          name: 'An extremely long long long    long long long name',
          age: 20,
        },
        {
          name: 'Lakshmi',
          age: 21,
        },
      ],
      selectedOptions: [],
    };
  },
  components: {
    VueMultipleSelect
  },
};
</script>

<style>
#app {
  /* margin: 20px 50px 40px 50px; */
}
</style>
