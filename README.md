<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <title>Perfil do Usuário</title>
  <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
  <style>
    body { margin: 0; font-family: Arial, sans-serif; background: #f4f6f8; color: #333; transition: background 0.3s, color 0.3s; }
    body.dark { background: #1e1e1e; color: #f4f6f8; }
    .navbar { background: #2c3e50; color: white; padding: 10px 20px; display: flex; justify-content: space-between; align-items: center; }
    .profile-menu { position: relative; display: inline-block; }
    .profile-pic { width: 45px; height: 45px; border-radius: 50%; object-fit: cover; cursor: pointer; border: 2px solid #fff; }
    .profile-status { position: absolute; bottom: 2px; right: 2px; width: 12px; height: 12px; border-radius: 50%; background: green; }
    .dropdown { display: none; position: absolute; right: 0; background: white; box-shadow: 0 2px 6px rgba(0,0,0,0.2); border-radius: 6px; min-width: 180px; z-index: 10; }
    .dropdown a { display: block; padding: 10px; text-decoration: none; color: #333; }
    .dropdown a:hover { background: #f1f1f1; }
    .profile-menu.open .dropdown { display: block; }
    .carousel { position: relative; overflow: hidden; width: 100%; max-width: 900px; margin: 20px auto; border-radius: 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.2); }
    .slide img { width: 100%; height: 350px; object-fit: cover; display: block; }
    .slide h3 { margin: 0; padding: 12px; font-size: 18px; background: rgba(0,0,0,0.6); color: #fff; position: absolute; bottom: 0; width: 100%; text-align: center; }
    .carousel button { position: absolute; top: 50%; transform: translateY(-50%); background: rgba(0,0,0,0.5); border: none; color: white; font-size: 24px; padding: 10px; cursor: pointer; }
    .carousel button:hover { background: rgba(0,0,0,0.7); }
    .carousel button:first-of-type { left: 10px; }
    .carousel button:last-of-type { right: 10px; }
    .news-container { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; padding: 20px; }
    .card { background: white; border-radius: 10px; box-shadow: 0 4px 12px rgba(0,0,0,0.15); overflow: hidden; transition: transform 0.2s; }
    body.dark .card { background: #2c2c2c; }
    .card:hover { transform: scale(1.03); }
    .card img { width: 100%; height: 180px; object-fit: cover; }
    .card-body { padding: 15px; text-align: center; }
    .card-body h3 { margin: 0; color: #2c3e50; font-size: 18px; }
    body.dark .card-body h3 { color: #f4f6f8; }
    .btn-read, .btn-fav, .btn-more { margin: 8px 5px; display: inline-block; padding: 8px 12px; background: #3498db; color: white; text-decoration: none; border-radius: 4px; cursor: pointer; }
    .btn-fav { background: #e74c3c; }
    .btn-read:hover, .btn-more:hover { background: #2980b9; }
    .btn-fav:hover { background: #c0392b; }
    .comments { margin-top: 10px; text-align: left; }
    .comments input { width: 80%; padding: 5px; margin-top: 5px; }
    .comments button { padding: 5px 10px; background: #27ae60; color: #fff; border: none; cursor: pointer; }
    .comments button:hover { background: #1e8449; }
    .more-content { margin-top: 10px; text-align: left; color: #555; }
    body.dark .more-content { color: #ddd; }
  </style>
</head>
<body :class="{dark: darkMode}">
  <div id="app">
    <div class="navbar">
      <h2>Game News</h2>
      <div class="profile-menu" :class="{ open: menuOpen }">
        <div style="position:relative; display:inline-block;">
          <img :src="profilePic" alt="Foto de Perfil" class="profile-pic" @click="toggleMenu">
          <span class="profile-status" :style="{ background: online ? 'green' : 'red' }"></span>
        </div>
        <div class="dropdown">
          <a href="#" @click="changePhoto">Alterar Foto</a>
          <a href="#" @click="toggleTheme">Mudar Tema</a>
          <a href="#">Ajuda</a>
          <a href="#" @click="logout">Sair</a>
        </div>
        <input type="file" ref="fileInput" style="display:none" @change="updatePhoto">
      </div>
    </div>

    <!-- Carrossel -->
    <div class="carousel" v-if="highlights.length">
      <div v-for="(slide, index) in highlights" 
           :key="slide.id" 
           class="slide" 
           v-show="currentSlide === index">
        <img :src="slide.image" :alt="slide.title">
        <h3>{{ slide.title }}</h3>
      </div>
      <button @click="prevSlide">◀️</button>
      <button @click="nextSlide">▶️</button>
    </div>

    <!-- Notícias -->
    <div class="news-container">
      <div v-for="news in gameNews" :key="news.id" class="card">
        <img :src="news.image" :alt="news.title">
        <div class="card-body">
          <h3>{{ news.title }}</h3>
          <button class="btn-read" @click="toggleComments(news)">💬 Comentários</button>
          <button class="btn-fav" @click="toggleFav(news)">❤ {{ news.favorites }}</button>
          <button class="btn-more" @click="toggleMore(news)">
            {{ news.showMore ? "Ler Menos" : "Ler Mais" }}
          </button>
          <div class="more-content" v-if="news.showMore">
            <p>{{ news.moreText }}</p>
          </div>
          <div class="comments" v-if="news.showComments">
            <p v-for="c in news.comments">{{ c.user }}: {{ c.text }}</p>
            <input v-model="newComment[news.id]" placeholder="Escreva um comentário...">
            <button @click="addComment(news)">Enviar</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <script>
  new Vue({
    el: "#app",
    data: {
      profilePic: "https://via.placeholder.com/120",
      menuOpen: false,
      darkMode: false,
      online: true,
      currentSlide: 0,
      highlights: [],
      gameNews: [],
      newComment: {},
      loggedUser: null
    },
    mounted() {
      const savedTheme = localStorage.getItem("darkMode");
      if (savedTheme) this.darkMode = JSON.parse(savedTheme);
      document.body.classList.toggle("dark", this.darkMode);

      this.loggedUser = JSON.parse(localStorage.getItem("loggedUser"));
      if (this.loggedUser) this.profilePic = this.loggedUser.photo || "https://via.placeholder.com/120";

      this.fetchHighlights();
      this.fetchNews();
    },
    methods: {
      toggleMenu() { this.menuOpen = !this.menuOpen; },
      changePhoto() { this.$refs.fileInput.click(); },
      updatePhoto(event) {
        const file = event.target.files[0];
        if (file) this.profilePic = URL.createObjectURL(file);
      },
      toggleTheme() {
        this.darkMode = !this.darkMode;
        document.body.classList.toggle("dark", this.darkMode);
        localStorage.setItem("darkMode", JSON.stringify(this.darkMode));
      },
      async toggleFav(news) {
        news.liked = !news.liked;
        news.favorites += news.liked ? 1 : -1;
        try {
          await axios.patch(`http://localhost:3000/news/${news.id}`, { favorites: news.favorites });
        } catch (err) { console.error(err); }
      },
      toggleComments(news) { news.showComments = !news.showComments; },
      toggleMore(news) { news.showMore = !news.showMore; },
      async addComment(news) {
        const commentText = this.newComment[news.id];
        if (!commentText || commentText.trim() === "") return;
        const comment = { user: this.loggedUser.name, text: commentText.trim() };
        news.comments.push(comment);
        this.newComment[news.id] = "";

        // Atualiza no JSON Server
        try {
          await axios.patch(`http://localhost:3000/news/${news.id}`, { comments: news.comments });
        } catch (err) { console.error(err); }
      },
      nextSlide() { this.currentSlide = (this.currentSlide + 1) % this.highlights.length; },
      prevSlide() { this.currentSlide = (this.currentSlide - 1 + this.highlights.length) % this.highlights.length; },
      logout() { window.location.href = "index.html"; },
      async fetchHighlights() {
        try {
          const res = await axios.get("http://localhost:3000/highlights");
          this.highlights = res.data;
        } catch (err) { console.error(err); }
      },
      async fetchNews() {
        try {
          const res = await axios.get("http://localhost:3000/news");
          this.gameNews = res.data.map(n => ({ ...n, showComments: false, showMore: false, liked: false }));
        } catch (err) { console.error(err); }
      }
    }
  });
  </script>
</body>
</html>
