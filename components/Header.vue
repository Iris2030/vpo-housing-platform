<script setup></script>
<template>
  <header class="site-header">
    <div class="container">
      <NuxtLink to="/" class="logo-wrapper">
        <img class="logo" src="../public/favicon.ico" alt="" />
        <span class="logo-name">VPO Housing</span>
      </NuxtLink>
      <nav class="nav-links">
        <NuxtLink to="/" exact-active-class="active">Головна</NuxtLink>
        <NuxtLink to="/search" exact-active-class="active">Пошук</NuxtLink>
        <NuxtLink to="/submit" exact-active-class="active">Додати</NuxtLink>
      </nav>
    </div>
  </header>
</template>

<style scoped lang="scss">
.site-header {
  background: #1e90ff;
  color: white;
  padding: 15px 0;
  // width: 100vw;

  .container {
    width: calc(100% - 40px);
    // margin: 0 auto;
    padding: 0 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  @media screen and (max-width: 460px) {
  .container {
    display: grid; 
    gap: 10px;
  }
 
  .nav-links a:first-child {
    margin-left: 0;
  }
}

  .logo-wrapper{
    display: flex;
    align-items: center;
  }
  .logo{
    width: 26px;
    height: 26px;
  }
  .logo-name {
    font-size: 20px;
    font-weight: bold;
    color: white;
    text-decoration: none;
    margin-left: 6px;
  }

  .nav-links a {
    margin-left: 20px;
    color: white;
    text-decoration: none;
    font-weight: 500;
  }

  .nav-links a.active {
    text-decoration: underline;
  }
}
</style>
