<template>
  <v-app>
    <v-app-bar color="primary" dark>
      <v-app-bar-title>
        <v-icon class="mr-2">mdi-grid</v-icon>
        Life App - Cellular Automata
      </v-app-bar-title>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <life-game />
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import LifeGame from '@/components/LifeGame.vue'
</script>

<style scoped>
.v-app-bar-title {
  display: flex;
  align-items: center;
}
</style>
