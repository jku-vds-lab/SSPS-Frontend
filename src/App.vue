<template>
  <v-app>
    <v-main>
      <ssps/>
    </v-main>
  </v-app>
</template>

<script>
import ssps from './components/ssps';

export default {
  name: 'App',

  components: {
    ssps,
  },

  data: () => ({
    //
  }),
};
</script>
