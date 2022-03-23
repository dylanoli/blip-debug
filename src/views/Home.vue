<template>
  <div class="home">
    <div style="display: flex">
      <v-text-field
        name="bot-key"
        label="Bot Key"
        v-model="key"
        outlined
        dense
        class="mr-5"
      ></v-text-field>
      <v-text-field
        name="number-user"
        label="User Id"
        v-model="userId"
        outlined
        dense
      ></v-text-field>
    </div>
    <v-btn @click="saveCredentials" class="mb-5">
      <v-icon class="mr-2">mdi-content-save</v-icon>Save credentials
    </v-btn>
    <br />
    <v-btn @click="getUserVariables" class="primary mr-5">
      <v-icon class="mr-2">mdi-account-search</v-icon>
      Get user variables
    </v-btn>
    <v-btn @click="getContextVariables" class="primary mr-5">
      <v-icon class="mr-2">mdi-nfc-search-variant</v-icon>
      Get context variables
    </v-btn>
    <v-btn class="mr-5" @click="clear">
      <v-icon class="mr-2">mdi-eraser-variant</v-icon>Clear
    </v-btn>
    <v-btn @click="() => (responseGetContextVariables = [])">
      <v-icon class="mr-2">mdi-refresh</v-icon>Resetar contato
    </v-btn>
    <br />
    <div class="mt-5 context">
      <!-- USER -->
      <div v-for="el in responseUserVariables" :key="el.key">
        <div v-if="el.key != 'extras'">
          <strong class="mr-1">{{ el.key }}:</strong>
          <span>{{ el.value }}</span>
        </div>
        <div v-else>
          <strong class="mr-1">{{ el.key }}:</strong>

          <div class="ml-10" v-for="el2 in el.value" :key="el2.key">
            <div>
              <strong class="mr-1">{{ el2.key }}:</strong>
              <span>{{ el2.value }}</span>
            </div>
          </div>
        </div>
      </div>
      <!-- CONTEXT -->
      <div v-for="(el, index) in responseGetContextVariables" :key="index">
        <strong class="mr-1">{{ el.key }}:</strong>
        <span>{{ el.value }}</span>
      </div>
    </div>

    <br />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { v4 as uuidv4 } from "uuid";
import { api } from "@/services";

const KEY_VALUE = "key-value";
const USER_ID_VALUE = "user-id-value";
const USER_VARIABEL_VALUE = "user-variabel-value";
@Component({
  components: {},
})
export default class Home extends Vue {
  key: string = "";
  userId: string = "";
  userVariabel: string = "";
  responseGetContextVariables = [];
  responseGetContextSpecifcVariabel = "";
  responseUserVariables: any[] = [];

  constructor() {
    super();

    const localKey = localStorage.getItem(KEY_VALUE);
    const localNumber = localStorage.getItem(USER_ID_VALUE);
    const localUserVariabel = localStorage.getItem(USER_VARIABEL_VALUE);
    if (localKey) this.key = localKey;
    if (localNumber) this.userId = localNumber;
    if (localUserVariabel) this.userVariabel = localUserVariabel;
  }
  saveCredentials() {
    localStorage.setItem(KEY_VALUE, this.key);
    localStorage.setItem(USER_ID_VALUE, this.userId);
    localStorage.setItem(USER_VARIABEL_VALUE, this.userVariabel);
  }
  clear() {
    this.responseGetContextVariables = [];
    this.responseUserVariables = [];
  }
  async getContextVariables() {
    this.clear();
    const body = {
      id: uuidv4(),
      to: "postmaster@msging.net",
      method: "get",
      uri: `/contexts/${this.userId}`,
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
      uri: `/contexts/${this.userId}/${key}`,
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
  async getUserVariables() {
    this.clear();
    const body = {
      id: uuidv4(),
      method: "get",
      uri: `/contacts/${this.userId}`,
    };

    let response = (
      (await api.post("commands", body, {
        headers: {
          Authorization: this.key,
        },
      })) as any
    ).data.resource;

    this.responseUserVariables = [];
    for (const key in response) {
      if (key != "extras")
        this.responseUserVariables.push({ key: key, value: response[key] });
      else {
        let extras: any[] = [];
        for (const keyj in response[key]) {
          extras.push({ key: keyj, value: response[key][keyj] });
        }
        this.responseUserVariables.push({ key: "extras", value: extras });
      }
    }
  }
}
</script>
<style scoped>
.home {
  text-align: left;
  padding: 50px;
}
strong {
  font-size: 10pt;
}
span {
  font-size: 10pt;
}
.context {
  height: 400px;
  overflow: auto;
}
</style>
