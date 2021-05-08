<template>
<div class="myPost">
  <h2>{{ postData.title }}</h2>
  <h4>{{ postData.content }}</h4>
</div>
</template>

<script>
export default {
  name: "Post",
  props: {
    postData: Object
  }
}
</script>

<style scoped>
.myPost {
  border: 2px solid black;
  padding: 5px;
  margin: 10px;
  display: inline-block;
  width: 200px;
  transition: 0.35s ease;
}
.myPost:hover {
  transform: scale(1.03);
}
</style>