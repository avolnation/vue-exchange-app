<template>
    <select v-model="selected" @change='$emit("sendData", createObjIdAndString($event.target.id, $event.target.value))'>
      <option v-for="(item, i) in exchangeData" :key="i">
        {{ item.currencyAbbr }}
      </option>
    </select>
</template>

<script>
export default {
  data() {
    return {
      selected: "",
    };
  },
  emits: ["sendData"],
  props: {
    exchangeData: Object,
  },
  computed: {},
  methods: {
      createObjIdAndString(id, string){
          let obj = {
              id: id,
              abbr: string
          };
          return obj;
      }
  },
};
</script>

<style>
select {
  font-size: 1.2rem;
  font-family: Google Sans;
  font-style: initial;
  color: #424242;
  background-color: #f7e7ce;
  width: 80px;
  height: 30px;
  border-radius: 5px;
  border-style: ridge;
  border: none;
  box-shadow: rgb(0 17 51 / 30%) 0px 3px 15px;
}
select:hover {
  cursor: pointer;
  background-color: #cfc1ab;
}
</style>