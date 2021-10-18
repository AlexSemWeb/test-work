<template>
  <v-app>
    <v-main>
      <MyForm />
    </v-main>
  </v-app>
</template>

<script>
import MyForm from "./components/MyFrom.vue";

export default {
  name: "App",
  components: {
    MyForm,
  },
};
</script>
