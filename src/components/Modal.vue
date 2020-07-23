<template>
  <div class="modal-container">
    <button id="modalTrigger" @click="open = true" v-if="!openByDefault">{{btnText}}</button>

    <div id="myModal" class="modal" v-if="open">
      <div class="modal-content">
        <span class="close" @click="open = false">&times;</span>
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Modal",
  props: {
    btnText: {
      type: String,
      required: true,
    },
    openByDefault: {
      type: Boolean,
      default: false,
    },
    parentData: {
      type: Object,
    },
  },
  data() {
    return {
      open: this.openByDefault,
    };
  },
};
</script>

<style>
.modal {
  position: fixed;
  z-index: 1;
  padding-top: 100px;
  left: 0;
  right: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgb(0, 0, 0); /* Fallback color */
  background-color: rgba(0, 0, 0, 0.4); /* Black w/ opacity */
}

/* Modal Content */
.modal-content {
  background-color: #fefefe;
  margin: auto;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
  border-radius: 0.3em;
  box-shadow: 10px 10px 30px rgba(0, 0, 0, 0.1);
  margin-top: 1em;
}

/* The Close Button */
.close {
  color: #aaaaaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
  margin-top: -7px;
}

.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}
</style>