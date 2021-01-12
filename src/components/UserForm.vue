<template>
  <div>
    <form class="form-container">
      <div>
        <span>name</span>
        <input
          v-model="form.name"
          @change="onNameChange"
          @input.native="onUserInputThrottle"
        />
      </div>
      <div>
        <span>phone</span>
        <input v-model="form.phone" @input="onUserInputThrottle" />
      </div>
      <div>
        <span>address</span>
        <input v-model="form.address" @input="onUserInputThrottle" />
      </div>
    </form>
    <button @click="onSave" :disabled="!saveButtonEnable">save</button>
  </div>
</template>

<script>
import axios from "axios";

const FORM_STORAGE_URL = "https://jsonbox.io/box_3db63994c93c2f38b947";

const throttle = (func, duration) => {
  let timer = null;
  return params => {
    console.log(params);
    if (timer) {
      // if called before duration
      clearTimeout(timer);
      timer = null;
    }
    timer = setTimeout(() => {
      func && func(params);
      timer = null;
    }, duration);
  };
};

export default {
  data() {
    return {
      initialForm: null,
      form: {
        name: "",
        phone: "",
        address: ""
      },
      saveButtonEnable: false
    };
  },
  methods: {
    onNameChange() {
      console.log("name change");
    },
    onSave() {
      console.log("userForm", this.form);
      // TODO: post form to: https://jsonbox.io/box_3db63994c93c2f38b947
      axios.post(FORM_STORAGE_URL, this.form);
      this.initialForm = { ...this.form };
      this.saveButtonEnable = false;
    },
    isFormTheSame(formOne, formTwo) {
      if (!formOne || !formTwo) return false;
      return (
        formOne.name === formTwo.name &&
        formOne.phone === formTwo.phone &&
        formOne.address === formTwo.address
      );
    },
    onFieldChange() {
      console.log("field change*********");
      const isFormChanged = !this.isFormTheSame(this.form, this.initialForm);
      this.saveButtonEnable = isFormChanged;
    },
    onUserInput(event) {
      console.log("user input: ", event);
      this.onFieldChange();
    }
  },
  async mounted() {
    // TODO: fetch form from https://jsonbox.io/box_3db63994c93c2f38b947
    // set initial form data
    console.log("component mounted");
    const { data: initialFormDataList } = await axios.get(FORM_STORAGE_URL);
    this.initialForm = initialFormDataList[0];
    this.form = {
      ...this.form,
      ...initialFormDataList[0]
    };
    console.log("form", this.form);

    this.onUserInputThrottle = throttle(this.onUserInput, 2000);
  }
};
</script>

<style scoped>
</style>