<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <title>Perfil - Socialink Games</title>
  <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      background: #f4f4f4;
    }
    .perfil-container {
      max-width: 900px;
      margin: 30px auto;
      background: #fff;
      border-radius: 10px;
      padding: 20px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    .perfil-header {
      display: flex;
      align-items: center;
      border-bottom: 2px solid #eee;
      padding-bottom: 20px;
    }
    .perfil-header img {
      width: 120px;
      height: 120px;
      border-radius: 50%;
      margin-right: 20px;
    }
    .perfil-header h2 {
      margin: 0;
      font-size: 24px;
    }
    .perfil-header p {
      color: #555;
      margin: 5px 0;
    }
    .perfil-estatisticas {
      display: flex;
      gap: 20px;
      margin-top: 10px;
    }
    .estatistica {
      font-size: 14px;
      color: #333;
    }
    .noticias {
      margin-top: 30px;
    }
    .noticia {
      display: flex;
      background: #fafafa;
      margin-bottom: 15px;
      border-radius: 8px;
      overflow: hidden;
      box-shadow: 0 0 5px rgba(0,0,0,0.1);
    }
    .noticia img {
      width: 200px;
      object-fit: cover;
    }
    .noticia-conteudo {
      padding: 15px;
    }
    .noticia-conteudo h3 {
      margin: 0;
      font-size: 18px;
    }
    .noticia-conteudo p {
      margin: 8px 0 0;
      color: #555;
    }
  </style>
</head>
<body>
  <div id="app" class="perfil-container">
    <div v-if="perfil" class="perfil-header">
      <img :src="perfil.avatar" alt="Avatar">
      <div>
        <h2>{{ perfil.nome }}</h2>
        <p>{{ perfil.bio }}</p>
        <div class="perfil-estatisticas">
          <div class="estatistica"><b>{{ perfil.seguidores }}</b> Seguidores</div>
          <div class="estatistica"><b>{{ perfil.seguindo }}</b> Seguindo</div>
        </div>
      </div>
    </div>

    <div class="noticias">
      <h2>Notícias Recentes</h2>
      <div v-for="noticia in perfil.noticias" :key="noticia.id" class="noticia">
        <img :src="noticia.imagem" alt="Notícia">
        <div class="noticia-conteudo">
          <h3>{{ noticia.titulo }}</h3>
          <p>{{ noticia.descricao }}</p>
        </div>
      </div>
    </div>
  </div>

  <script>
    new Vue({
      el: "#app",
      data: {
        perfil: null
      },
      created() {
        axios.get("http://localhost:3000/perfil")
          .then(res => {
            this.perfil = res.data
          })
          .catch(err => console.error(err))
      }
    })
  </script>
</body>
</html>
