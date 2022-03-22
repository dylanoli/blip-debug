<template>
  <div class="home">
    <strong style="margin-right: 5px">Bot key</strong>
    <input v-model="key" type="text" placeholder="bot key" />
    <br />
    <strong style="margin-right: 5px">Number user</strong>
    <input v-model="numberUser" type="text" placeholder="Number user" />
    <br />
    <button @click="saveCredentials">Save credentials</button>
    <br />
    <br />
    <button @click="getContextVariables">Get context variables</button>
    <button @click="() => (responseGetContextVariables = [])">Clear</button>
    <br />
    <div v-for="(el, index) in responseGetContextVariables" :key="index">
      <strong>{{ el.key }}:</strong>
      <span>{{ el.value }}</span>
    </div>
    <br />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { v4 as uuidv4 } from "uuid";
import { api } from "@/services";

const KEY_VALUE = "key-value";
const NUMBER_USER_VALUE = "number-user-value";
const USER_VARIABEL_VALUE = "user-variabel-value";
@Component({
  components: {},
})
export default class Home extends Vue {
  key: string = "";
  numberUser: string = "";
  userVariabel: string = "";
  responseGetContextVariables = [];
  responseGetContextSpecifcVariabel = "";

  constructor() {
    super();

    const localKey = localStorage.getItem(KEY_VALUE);
    const localNumber = localStorage.getItem(NUMBER_USER_VALUE);
    const localUserVariabel = localStorage.getItem(USER_VARIABEL_VALUE);
    if (localKey) this.key = localKey;
    if (localNumber) this.numberUser = localNumber;
    if (localUserVariabel) this.userVariabel = localUserVariabel;
  }
  saveCredentials() {
    localStorage.setItem(KEY_VALUE, this.key);
    localStorage.setItem(NUMBER_USER_VALUE, this.numberUser);
    localStorage.setItem(USER_VARIABEL_VALUE, this.userVariabel);
  }
  async getContextVariables() {
    const body = {
      id: uuidv4(),
      to: "postmaster@msging.net",
      method: "get",
      uri: `/contexts/${this.numberUser}@wa.gw.msging.net`,
    };
    const response = (
      (await api.post("commands", body, {
        headers: {
          Authorization: this.key,
        },
      })) as any
    ).data.resource.items;

    this.responseGetContextVariables = await response.map((element: any) => ({
      key: element,
      value: "searching...",
    }));

    this.responseGetContextVariables.forEach((element: any, index: number) => {
      this.getContextSpecificVariabel(index);
    });
  }

  async getContextSpecificVariabel(index: number) {
    const key = (this.responseGetContextVariables[index] as any).key;
    const body = {
      id: uuidv4(),
      to: "postmaster@msging.net",
      method: "get",
      uri: `/contexts/${this.numberUser}@wa.gw.msging.net/${key}`,
    };

    let response = (
      (await api.post("commands", body, {
        headers: {
          Authorization: this.key,
        },
      })) as any
    ).data.resource;

    const value = response ? response : "[[error]]";
    this.$set(this.responseGetContextVariables, index, { key, value });
  }
}
</script>
<style scoped>
.home {
  text-align: left;
}
</style>
