<template>
  <div class="background">
    <img src="https://images.pexels.com/photos/1698618/pexels-photo-1698618.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1" />
  </div>
  <div class="grid-container">
    <div class="grid-x grid-padding-x">
      <div class="medium-6 cell get lab">
        <label>Judul
          <input v-model="title" type="text" placeholder="Masukkan judul">
        </label>
      </div>
      <div class="medium-6 cell get lab">
        <label>Komentar
          <input v-model="comment" type="text" placeholder="Masukkan komentar">
        </label>
      </div>
    </div>
    <button @click="saveComment" class="button">Save</button>

    <div class="grid-x grid-padding-x">
      <div v-for="comment in comments" :key="comment.id" class="medium-6 cell">
        <div class="card" style="background-color: transparent;">
          <div class="card-divider" style="color: white; background-color: rgba(128, 128, 128, 0.5);">
            {{ comment.title }}
          </div>
          <div class="card-section">
            <p style="color: white;">{{ comment.comment }}</p>
            <button @click="editComment(comment)" class="btn button success small">Edit</button>
            <button @click="deleteComment(comment.id)" class="btn button alert small">Delete</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const title = ref('');
const comment = ref('');
const comments = ref([]);
const editMode = ref(false);
const editId = ref(null);

const fetchComments = async () => {
  try {
    const response = await axios.get('http://localhost:3000/comments');
    comments.value = response.data;
  } catch (error) {
    console.error('Error fetching comments:', error);
  }
};

const saveComment = async () => {
  if (editMode.value) {
    updateComment();
  } else {
    try {
      const newComment = {
        title: title.value,
        comment: comment.value
      };
      const response = await axios.post('http://localhost:3000/comments', newComment);
      comments.value.push(response.data);
      resetForm();
    } catch (error) {
      console.error('Error saving comment:', error);
    }
  }
};

const deleteComment = async (id) => {
  try {
    await axios.delete(`http://localhost:3000/comments/${id}`);
    comments.value = comments.value.filter(comment => comment.id !== id);
  } catch (error) {
    console.error('Error deleting comment:', error);
  }
};

const editComment = (comment) => {
  title.value = comment.title;
  comment.value = comment.comment;
  editId.value = comment.id;
  editMode.value = true;
};

const updateComment = async () => {
  try {
    const updatedComment = {
      title: title.value,
      comment: comment.value
    };
    await axios.put(`http://localhost:3000/comments/${editId.value}`, updatedComment);
    comments.value = comments.value.map(comment =>
      comment.id === editId.value ? updatedComment : comment
    );
    resetForm();
  } catch (error) {
    console.error('Error updating comment:', error);
  }
};

const resetForm = () => {
  title.value = '';
  comment.value = '';
  editMode.value = false;
  editId.value = null;
};

onMounted(fetchComments);
</script>

<style>
.lab label{
  color: white;
  font-weight: bold;
}
.lab input{
  background-color: transparent;
  backdrop-filter: blur(5px);
}
.card {
  margin: 10%;
  backdrop-filter: blur(5px);
}
.card-section button {
  display: table;
  margin: 10px;
}
.background img{
  position: fixed;
  top: 0;
  left: 0;
  min-height: 90%;
  min-width: 1500px;
  width: 100%;
  height: 100%;
  overflow: hidden;
  z-index: -2;
  object-fit: cover;
  -webkit-background-size:cover; -moz-background-size:cover; -o-background-size:cover; background-size: cover;
  filter: brightness(0.8);
}
</style>
