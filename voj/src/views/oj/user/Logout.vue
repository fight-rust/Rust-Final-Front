<template>
</template>

<script>
import api from '@/common/api'
import store from "@/store";

export default {
  mounted() {
    api.logout().then(res => {
      this.$store.dispatch('clearUserInfoAndToken')

      this.$router.replace({
        path: '/home'
      })
    })
  }
}
</script>
