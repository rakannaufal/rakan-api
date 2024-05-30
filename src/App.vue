<template>
  <div class="container">
    <h2>Integrasi API</h2>
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" /> <br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit" class="save">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="edit(article)">Edit</button>
        <button @click="remove(article.id)">Delete</button>
      </li>
    </ul>
    <button @click="load" class="load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

export default {
  setup() {
    const form = reactive({
      id: null,
      title: "",
      content: "",
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get("http://localhost:3000/articles");
        articles.value = response.data;
      } catch (error) {
        console.error("Error loading articles:", error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : "http://localhost:3000/articles";
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, {
          title: form.title,
          content: form.content,
        });

        if (form.id) {
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
        console.error("Error saving article:", error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (error) {
        console.error("Error deleting article:", error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 3rem;
  background-color: #bed7dc;
  border-radius: 7px;
}

.form {
  margin-bottom: 20px;
}

.form input,
.form textarea {
  width: 700px;
  padding: 15px;
  color:#fff;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f9f9f9;
}

.form button {
  width: 100%;
  padding: 10px 15px;
  background-color: #f1eedc;
  border: none;
  color: #000000;
  font-weight: bold;
  border-radius: 4px;
  cursor: pointer;
}

.form button:hover {
  background-color: #d3cfb6;
}

.article-list {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  padding: 0;
  list-style-type: none;
  color: #000;
}

.article-item {
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f9f9f9;
}

.article-item h3 {
  margin: 0 0 10px;
}

.article-item p {
  margin: 0 0 10px;
}

.article-item button {
  padding: 5px 10px;
  margin-right: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.article-item button:hover {
  opacity: 0.8;
}

.article-item button:first-of-type {
  background-color: #007bff;
  color: white;
}

.article-item button:last-of-type {
  background-color: #dc3545;
  color: white;
}

.load-button {
  width: 100%;
  padding: 10px 15px;
  background-color: #9ac9d8;
  border: none;
  color: rgb(3, 3, 3);
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}

.load-button:hover {
  background-color: #7eb1c2;
}

h2 {
  font-size: 2rem;
  color: #000;
}
</style>
