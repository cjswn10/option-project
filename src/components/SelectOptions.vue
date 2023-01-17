<template>
  <div v-for="(item, idx) in groupList" :key="idx">
    <select v-model="selectedOption[idx]" :disabled="item.disabled" @change="selectChange(idx)">
      <option value="" disabled>{{ item.title }}</option>
      <option v-for="op in item.options" :key="op" :value="op" :disabled="calcRemainCount(op, idx) == '(품절)'">{{ op }}<span> {{ calcRemainCount(op, idx) }}</span></option>
    </select>
  </div>
  <div>{{ selectedResult }}</div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      groupList: [],
      selectedOption: []
    }
  },
  computed: {
    selectedResult() {
      return this.selectedOption.filter(v => v).join(' / ');
    }
  },
  created() {
    this.fetchOptions('http://18.183.198.182:3333/api/621f2588d8d85b8d78eb3e64')
  },
  methods: {
    async fetchOptions(apiUrl) {

      await axios.get(apiUrl).then(response => {

        this.groupList = response?.data?.data?.groupList;
        this.countList = response?.data?.data?.countList;

        this.groupList.forEach((obj, index) => {
          obj.index = index;
          obj.disabled = index ? true : false;
          this.selectedOption[obj.index] = '';
        });

      }).catch(error => {
        console.log(error);
      })
    },
    calcRemainCount(op, idx) {

      let text = '';
      const selectedArr = this.selectedOption.slice(0, idx).concat(op);
      const cnt = this.countList.reduce((acc, cur) => {
          return acc += selectedArr.every(v => cur.combination.includes(v)) ? cur.remainCount : 0; 
        }, 0);

      if (idx < this.groupList.length - 1) {
        text = cnt == 0 ? `(품절)` : ``;
      } else {
        text = cnt == 0 ? `(품절)` : `(${cnt}개 구매가능)`;
      }

      return text;
    },
    selectChange(idx) {
      if (idx < this.groupList.length - 1) {
        this.selectedOption.fill('', idx + 1);
        this.groupList[++idx].disabled = false;
        while (++idx < this.groupList.length) {
          this.groupList[idx].disabled = true;
        }
      }
    }
  }
}
</script>

<style>
select {
  width: 250px;
  height: 40px;
  font-size: 16px;
  padding: 10px;
}
</style>