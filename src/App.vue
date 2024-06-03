<template>
  <div class="container">
    <header class="header">
      <h1>Artikel Saya</h1>
    </header>
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="button edit-button">Edit</button>
        <button @click="remove(article.id)" class="button delete-button">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: "",
      content: ""
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      if (!form.title || !form.content) {
        alert('Title and Content cannot be empty.');
        return;
      }

      try {
        const url = form.id ? `http://localhost:3000/articles/${form.id}` : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';

        if (!form.id) {
          form.id = Math.random().toString(36).substr(2, 9);
        }

        const response = await axios[method](url, form);

        if (form.id && method === 'put') {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return {
      form,
      articles,
      save,
      edit,
      remove
    };
  }
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Roboto', sans-serif;
  background: linear-gradient(135deg, #65f668 0%, #85fde9 100%);
  border-radius: 20px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
}

.header {
  text-align: center;
  margin-bottom: 20px;
}

.header h1 {
  margin: 0;
  font-size: 3em;
  color: #fff;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.form {
  margin-bottom: 20px;
  padding: 20px;
  border-radius: 10px;
  background-color: rgba(255, 255, 255, 0.9);
}

.input, .textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: none;
  border-radius: 4px;
  background-color: rgba(255, 255, 255, 0.7);
  font-family: 'Arial', sans-serif; /* Apply 'Open Sans' font */
}

.input:focus, .textarea:focus {
  outline: none;
  background-color: #fff;
}

.button {
  padding: 10px 20px;
  margin-right: 10px;
  border: none;
  border-radius: 4px;
  background-color: #4CAF50;
  color: #fff;
  cursor: pointer;
}

.button:hover {
  background-color: #45a049;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  border-radius: 10px;
  padding: 20px;
  margin-bottom: 10px;
  background-color: rgba(255, 255, 255, 0.9);
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.article-item:hover {
  transform: translateY(-5px);
}

.article-item h3 {
  margin: 0 0 10px;
  color: #333;
}

.article-item p {
  margin: 0 0 10px;
  color: #666;
}

.edit-button, .delete-button, .load-button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  color: #fff;
  cursor: pointer;
}

.edit-button {
  background-color: #2196F3;
}

.delete-button {
  background-color: #f44336;
}

.load-button {
  background-color: #555;
}

.edit-button:hover, .delete-button:hover, .load-button:hover {
  opacity: 0.8;
}
</style>
