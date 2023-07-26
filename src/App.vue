<template>
  <div class="app">
    <div class="header container h-100 p-5">
      <h1 class="mb-5">CarWash Cerca De Mi</h1>
      <div class="d-flex justify-content-center h-100">
        <div class="searchbar w-50 mx-2">
          <input type="text" class="input form-control" placeholder="Enter a City...">
        </div>
        <button class="btn-search btn btn-primary">Search <i class="fas fa-search"></i></button>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  }
}
</script>

<style>
body{
  background-color: #121212 !important;
}
.header{
  background-color: #212730;
  border-radius: 20px;
  color: #fff;
  text-align: center;
  font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
  margin-top: 5rem;
}
.btn-search{
  background-image: linear-gradient(to right, lavender, teal);
}
</style>
