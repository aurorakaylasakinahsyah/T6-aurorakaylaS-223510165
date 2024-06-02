<template>
  <div class="container">
    <h1 class="title">DRAMA INDO</h1>

    <!-- Form untuk input artikel -->
    <form class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="button" @click="saveForm" class="button save">Save</button>
    </form>

    <!-- Tabel untuk menampilkan artikel -->
    <table class="article-table">
      <thead>
        <tr>
          <th>Title</th>
          <th>Content</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="article in articles" :key="article.id" :class="{ 'user-input': !article.fromDb }" class="article-row">
          <td>{{ article.title }}</td>
          <td>{{ article.content }}</td>
          <td>
            <button @click="editArticle(article)" class="button edit">Edit</button>
            <button @click="deleteArticle(article.id)" class="button delete">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Tombol untuk memuat data -->
    <button @click="loadArticles" class="button load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = ref({
      id: null,
      title: '',
      content: '',
    });
    const articles = ref([]);

    async function loadArticles() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data.map(article => ({ ...article, fromDb: true }));
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function saveForm() {
      try {
        const url = form.value.id ? `http://localhost:3000/articles/${form.value.id}` : "http://localhost:3000/articles";
        const method = form.value.id ? "put" : "post";
        const response = await axios[method](url, {
          title: form.value.title,
          content: form.value.content,
        });

        if (form.value.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        form.value.id = null;
        form.value.title = "";
        form.value.content = "";
      } catch (error) {
        console.error("Error saving article:", error);
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function editArticle(article) {
      form.value.id = article.id;
      form.value.title = article.title;
      form.value.content = article.content;
    }

    onMounted(loadArticles);

    return { form, articles, saveForm, deleteArticle, editArticle, loadArticles };
  }
};
</script>

<style scoped>
.container {
  margin: 0 auto;
  max-width: 800px;
  padding: 20px;
  background-color: #fce4ec; /* Light pink background color */
}

.title {
  text-align: center;
  font-size: 2em;
  margin-bottom: 20px;
}

.form {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.input, .textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #cccccc;
  border-radius: 5px;
}

.button {
  padding: 10px 20px;
  margin: 5px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.button.save {
  background-color: #4CAF50;
  color: white;
}

.button.edit {
  background-color: #008CBA;
  color: white;
}

.button.delete {
  background-color: #f44336;
  color: white;
}

.button.load {
  background-color: #555555;
  color: white;
  display: block;
  margin: 20px auto;
}

.article-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.article-table th, .article-table td {
  padding: 15px;
  border: 1px solid #cccccc;
  text-align: left;
}

.article-table th {
  background-color: #f2f2f2;
}

.article-row.user-input {
  background-color: #ffcc80; /* Light orange background for user input */
}

.article-row:nth-child(even) {
  background-color: #e0e0e0; /* Slightly darker gray background for even items */
}

.article-row strong {
  font-size: 1.2em;
}
</style>
