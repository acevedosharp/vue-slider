<template>
  <div id="app">
    <h1>vue-slider</h1>
    <Slider :step="1" :do-auto-move="true" :auto-move-period="5100">
      <Post v-for="post in posts"
            :key="post.id"
            :post-data="post"/>
    </Slider>
  </div>
</template>

<script>
import Slider from './components/Slider'
import Post from './components/Post'

export default {
  name: 'App',
  components: {
    Slider,
    Post
  },
  data() {
    const singlePost = {
      title: "Joe",
      content: "Mama"
    }
    const posts = []
    for (let i = 1; i <= 24; i++) {
      posts.push({
        id: i,
        title: `${singlePost.title} #${i}`,
        content: singlePost.content
      })
    }
    return {
      posts
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
