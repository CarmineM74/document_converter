<template>
  <div id="app" class="container mx-auto bg-yellow">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style lang="scss">
  @tailwind preflight;
  @tailwind components;
  @tailwind utilities;
</style>
