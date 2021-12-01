<script>
export default {
  props: {
    exchangeData: Object,
  },
  data() {
    return {};
  },
  name: "Modal",
  methods: {
    close() {
      this.$emit("close");
    },
  },
};
</script>

<template>
  <transition name="modal-fade">
    <div class="modal-backdrop" @click.self="close">
      <div
        class="modal"
        role="dialog"
        aria-labelledby="modalTitle"
        aria-describedby="modalDescription"
      >
        <div class="availableCurrencies">Available Currencies List</div>
        <hr />
        <div id="dataWrapper">
          <p class="currencyplaceholder" v-for="(item, i) in exchangeData" :key="i">
            {{ `${item.currencyAbbr} / ${item.currencyName}` }}
          </p>
        </div>
      </div>
    </div>
  </transition>
</template>

<style>
::-webkit-scrollbar {
  width: 0px; /* ширина scrollbar'a */
  background: transparent; /* опционально */
}
.currencyplaceholder {
  font-size: 0.9em;
  color: #f7e7ce;
  background-color: #EC8244;
  border-radius: 2px;
  margin: 5px;
  padding: 0.5em;
  width: 40%;
}
.modal-backdrop {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
}
.availableCurrencies {
  display: flex;
  justify-content: center;
}
.modal {
  background: #424242;
  padding: 0.7em;
  max-height: 50%;
  max-width: 400px;
  border-radius: 20px;
  box-shadow: 2px 1px 10px 0px #424242;
  overflow: auto;
}

#dataWrapper {
  padding-top: 10px;
  color:black;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.btn-close {
  position: absolute;
  top: 0;
  right: 0;
  border: none;
  font-size: 20px;
  padding: 10px;
  cursor: pointer;
  font-weight: bold;
  color: #4aae9b;
  background: transparent;
}

.btn-green {
  color: white;
  background: #4aae9b;
  border: 1px solid #4aae9b;
  border-radius: 2px;
}

.modal-fade-enter,
.modal-fade-leave-to {
  opacity: 0;
}

.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.3s ease-in-out;
}
</style>