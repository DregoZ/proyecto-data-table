<template>
  <v-app>
    <TablaParticipantes></TablaParticipantes>
  </v-app>
</template>

<script>
/*import HelloWorld from './components/HelloWorld';*/
import TablaParticipantes from "./components/TablaParticipantes";
export default {
  name: "App",

  components: {
    TablaParticipantes
  },

  data: () => ({
    //
  })
};
</script>
