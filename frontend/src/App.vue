<template>
  <div id="app">
    <nav class="px-4 d-flex align-items-center">
      <div id="essist">ESSIST</div>
      <router-link to="/store">Storefront</router-link>
      <span>|</span>
      <router-link to="/stock">Stock Management</router-link>
      <span>|</span>
      <router-link to="/history">Order History</router-link>
      <span>|</span>
      <router-link to="/category">Category Management</router-link>
      <!-- <span>|</span>
      <router-link to="/pdf">PDFcontent</router-link> -->
    </nav>
    <router-view class="router-view" />
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
}

nav {
  /* padding: 15px 30px; */
  height: 12vh;
  text-align: center;
  background-color: white;
  box-shadow: 0px 0px 15px #8997a6;
  position: sticky;
  top: 0;
  z-index: 3;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}

#essist {
  font-size: 3em;
  font-weight: bold;
  padding: 0 30px 0 0;
}

nav span {
  padding: 0 10px;
}

.router-view {
  background-color: #f5f5f5;
  min-height: 88vh;
}
.test {
  border: 1px solid blue;
}
.bold {
  font-weight: bold;
}
.item-card {
  width: 320px;
}
.outer {
  position: relative;
}
</style>
