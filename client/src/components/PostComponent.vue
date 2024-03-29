<template>
  <div class = "container">
    <div class="options">
      <input class="search" type="text" v-model="search" placeholder="Search by location">
      <button class="form" @click="showForm = !showForm"><i class="fas fa-caret-down"></i></button>
    </div>
    <form @submit.prevent="createPost" v-if="showForm" class="create-post">
      <div class="form-group">
        <label for="create-sign1">Sign 1</label>
        <input type="number" id="create-sign1" v-model.number="sign1" placeholder="Number of cars">
      </div>
      <div class="form-group">
        <label for="create-sign2">Sign 2</label>
        <input type="number" id="create-sign2" v-model.number="sign2" placeholder="Number of cars">
      </div>
      <div class="form-group">
        <label for="create-total">Total</label>
        <input type="number" id="create-total" :value="total" placeholder="Total number of cars" readonly>
      </div>
      <div class="form-group">
        <label for="create-start">Start</label>
        <input type="datetime-local" id="create-start" v-model="startTime">
      </div>
      <div class="form-group">
        <label for="create-end">End</label>
        <input type="datetime-local" id="create-end" v-model="endTime">
      </div>
      <div class="form-group">
        <label for="create-duration">Duration</label>
        <input type="string" id="create-duration" :value="displayTotalTime" placeholder="Duration" readonly>
      </div>
      <div class="form-group">
        <label for="create-location">Location</label>
        <input type="text" id="create-location" v-model="location" placeholder="Location">
      </div>
      <div class="form-group">
        <button type="submit">Submit</button>
      </div>
    </form>
    <hr>
    <p class="error" v-if="error">{{ error }}</p>
    <div v-if="loading">Loading...</div>
    <div class = "posts-container" v-else>
      <div class="post"
        v-for="(post, index) in filteredPosts"
        v-bind:item="post"
        v-bind:index="index"
        v-bind:key="post._id">
        <button class ="delete-button" v-on:click="deletePost(post._id)"><i class="fas fa-trash"></i></button>
        <p class="date">
          {{
            post.start ? post.start.split('T')[0] : ''
            /*(new Date(post.start).getMonth() + 1) + '/' +
            new Date(post.start).getDate() + '/' +
            new Date(post.start).getFullYear()*/
          }}
        </p>
        <div class="grid-container">
          <div> 
            <div>
              <p><span class="category">Sign 1: </span> <span> {{ post.sign1 }} </span></p>
              <!-- <p class="value">{{ post.sign1 }}</p> -->
            </div>

            <div>
              <p><span class="category">Sign 2: </span> <span> {{ post.sign2 }} </span></p>
              <!-- <p class="value">{{ post.sign2 }}</p> -->
            </div>

            <div>
              <p><span class="category">Total: </span> <span> {{ post.total }} </span></p>
              <!-- <p class="value">{{ post.total }}</p> -->
            </div>
          </div>
          
          <div> 
            <div>
              <p><span class="category">Start: </span> <span> {{ post.start.split('.')[0] }} </span></p>
            </div>

            <div>
              <p><span class="category">End: </span> <span> {{ post.end.split('.')[0] }} </span></p>
            </div>

            <div>
              <p><span class="category">Duration: </span> <span> {{ post.duration }} </span></p>
            </div>
          </div>

          <div>
            <p><span class="category">Location: </span> <span> {{ post.location }} </span></p>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
import PostService from '../PostService';
import { useToast } from 'vue-toastification';
const toast = useToast();

export default {
  name: 'PostComponent',
  data() {
    return {
      search: '',
      posts: [],
      error: '',
      startTime: '',
      endTime: '',
      displayTotalTime: '',
      location: '',
      sign1: 0,
      sign2: 0,
      total: 0,
      loading: false,
      showForm: false, // New data property to control form visibility
    }
  },
  async created() {
    this.loading = true;
    try {
      const posts = await PostService.getPosts();
      this.posts = posts.sort((a, b) => new Date(b.start) - new Date(a.start));
    } catch (err) {
      this.error = err.message;
      toast.error('Failed to load posts.');
    } finally {
      this.loading = false;
    }
  },
  computed: {
    filteredPosts() {
      return this.posts.filter(post => {
        return post.location.toLowerCase().includes(this.search.toLowerCase());
      });
    },
    calculatedTotal() {
      return this.sign1 + this.sign2;
    },
    calculatedDuration() {
      const duration = new Date(this.endTime) - new Date(this.startTime);

      const seconds = Math.floor(duration / 1000);
      const hours = Math.floor(seconds / 3600).toString().padStart(2, '0');
      const minutes = Math.floor((seconds % 3600) / 60).toString().padStart(2, '0');
      const remainingSeconds = (seconds % 60).toString().padStart(2, '0');

      return `${hours}:${minutes}:${remainingSeconds}`;
    },
  },
  watch: {
    calculatedTotal(newTotal) {
      this.total = newTotal;
    },
    calculatedDuration(newDuration) {
      this.displayTotalTime = newDuration;
    },
  },
  methods: {
    async createPost() {
      try {
        await PostService.insertPost(this.sign1, this.sign2, this.total, new Date(this.startTime), new Date(this.endTime), this.displayTotalTime, this.location);
        const posts = await PostService.getPosts();
        this.posts = posts.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
        toast.success('New post added.');
      } catch (error) {
        toast.error('Failed to add new post.');
      }
      this.sign1 = 0;
      this.sign2 = 0;
      this.total = 0;
      this.startTime = '';
      this.endTime = '';
      this.displayTotalTime = '';
      this.location = '';
  },
    async deletePost(id) {
      try {
        await PostService.deletePost(id);
        const posts = await PostService.getPosts();
        this.posts = posts.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
        toast.success('Post deleted.');
      } catch (error) {
        toast.error('Failed to delete post.');
      }
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div.container {
  margin: 0 auto;
  width: 50%;
}
p.error {
  border: 1px solid red;
  background-color: pink;
  padding: 10px;
  margin-bottom: 15px;
}



div.post{
  position: relative;
  text-align: left;
  display: grid;
  background-color: white;
  box-shadow: 2px 2px 12px 1px rgba(140,140,140,.5);
  padding: 20px;
  margin: 15px;
  margin-bottom: 30px;
}
.delete-button {
  position: absolute;
  top: 0;
  right: 0;
  margin: 5px;
  background-color: rgb(201, 201, 201);
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
  float: right;
}
.delete-button:hover {
  background-color: rgb(150, 150, 150);
}

div.created-at {
  position: absolute;
  bottom: 0;
  right: 0;
  font-size: 10px;
  color: grey;
  padding: 5px;
  background-color: white;
  border: 1px solid grey;
  border-radius: 5px;
}

p.text {
  font-size: 20px;
  color: black;
  margin-bottom: 10px;
}
.date {
  /*background-color: rgb(236, 236, 236);*/
  text-align: center;
  font-size: 24px;
  font-weight: bold;
  margin: 0;
  padding: 0;
}
.category {
  font-size: 20px;
  font-weight: bold;
}
.search {
  padding: 10px;
  width: 100%;
  margin: 20px 0;
  margin-right: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.options {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.form {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  margin: 20px 0;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.form:hover {
  background-color: #1f8f2f;
}
.create-post {
  display: grid;
  grid-template-columns: 1fr; /* Add this line */
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f8f8f8;
}

.create-post .form-group {
  display: grid; /* Add this line */
  grid-template-columns: 1fr; /* Add this line */
  margin-bottom: 20px;
}

.create-post .form-group label {
  display: block;
  margin-bottom: 10px;
}

.create-post .form-group input {
  width: 100%;
  max-width: 95%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.create-post button {
  max-width: 50%; /* Ensure it doesn't overflow its container */
  margin: auto; /* Center the button */
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #007BFF;
  color: white;
  cursor: pointer;
}
.create-post button:hover {
  background-color: #0056b3;
}
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 10px;
}

.vertical-line {
  border-left: 1px solid black;
  height: 50px;
}
</style>
