<template>
  <app-header />

  <router-view v-slot="{ Component }">
    <transition name="fade" mode="out-in">
      <component :is="Component"></component>
    </transition>
  </router-view>

  <!-- Player -->
  <Player></Player>

  <auth-modal></auth-modal>

</template>

<script>
import Player from './components/Player'
import AppHeader from './components/Header.vue';
import AuthModal from './components/Auth.vue';

export default {
  name: 'App',
  components: {
    Player,
    AppHeader,
    AuthModal,
  },
  created(){
    this.$store.dispatch('init_login');
  }
};
</script>
<style>
  .fade-enter-form{
    opacity: 0;
  }
  .fade-enter-active{
    transition:all 0.5s linear;
  }
  .fade-leave-to{
    transition:all 0.5s linear;
    opacity:0;
  }
</style>
