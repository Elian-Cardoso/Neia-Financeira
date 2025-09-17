<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <title>Cadastro e Login</title>
  <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; margin: 40px; background: #f9f9f9; }
    h1 { text-align: center; color: #333; }
    form { max-width: 400px; margin: auto auto 20px auto; padding: 20px; background: #fff; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); display: flex; flex-direction: column; }
    input, button { margin: 5px 0; padding: 10px; font-size: 16px; }
    button { cursor: pointer; background: #3498db; color: #fff; border: none; border-radius: 4px; }
    button:hover { background: #2980b9; }
    .message { text-align: center; margin-bottom: 20px; font-size: 16px; }
    .success { color: green; }
    .error { color: red; }
    .toggle { text-align: center; margin-bottom: 20px; }
    .toggle button { margin: 0 5px; }
    .search { display: block; margin: 10px auto; padding: 10px; max-width: 400px; width: 100%; }
    ul { list-style: none; padding: 0; max-width: 600px; margin: 20px auto; }
    li { background: #fff; margin: 5px 0; padding: 10px; border-radius: 4px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); display: flex; justify-content: space-between; align-items: center; }
    .admin-btn { margin-left: 5px; padding: 5px 8px; border-radius: 4px; color: #fff; border: none; cursor: pointer; }
    .edit-btn { background: #f39c12; }
    .save-btn { background: #27ae60; }
    .delete-btn { background: #e74c3c; }
    .sort-btn { display: block; margin: 10px auto; padding: 10px 15px; background: #2ecc71; color: #fff; border: none; border-radius: 4px; cursor: pointer; }
    .sort-btn:hover { background: #27ae60; }
  </style>
</head>
<body>
<div id="app">
  <h1>Cadastro e Login</h1>

  <!-- Alternar -->
  <div class="toggle">
    <button @click="mode = 'register'">Cadastrar</button>
    <button @click="mode = 'login'">Entrar</button>
  </div>

  <!-- Mensagem -->
  <div class="message" v-if="message" :class="messageType">{{ message }}</div>

  <!-- Cadastro -->
  <form v-if="mode === 'register'" @submit.prevent="addUser">
    <input type="text" v-model="newUser.name" placeholder="Nome" required>
    <input type="email" v-model="newUser.email" placeholder="Email" required>
    <input type="password" v-model="newUser.password" placeholder="Senha" required>
    <input type="text" v-model="newUser.phone" placeholder="Telefone">
    <input type="number" v-model="newUser.age" placeholder="Idade">
    <button type="submit">Cadastrar</button>
  </form>

  <!-- Login -->
  <form v-if="mode === 'login'" @submit.prevent="loginUser">
    <input type="email" v-model="loginEmail" placeholder="Email" required>
    <input type="password" v-model="loginPassword" placeholder="Senha" required>
    <button type="submit">Entrar</button>
  </form>

  <!-- Admin -->
  <div v-if="isAdmin">
    <input type="text" v-model="searchQuery" class="search" placeholder="Pesquisar usuário...">
    <button class="sort-btn" @click="sortUsers">Ordenar por Nome</button>
  </div>

  <!-- Lista -->
  <ul v-if="isAdmin">
    <li v-for="user in filteredUsers" :key="user.id">
      <span v-if="!user.editing">{{ user.name }} - {{ user.email }}</span>
      <span v-else>
        <input v-model="user.name">
        <input v-model="user.email">
        <input v-model="user.phone">
        <input v-model="user.age" type="number">
        <input v-model="user.password" type="password">
      </span>
      <div>
        <button v-if="!user.editing" class="admin-btn edit-btn" @click="user.editing = true">Editar</button>
        <button v-else class="admin-btn save-btn" @click="updateUser(user)">Salvar</button>
        <button class="admin-btn delete-btn" @click="deleteUser(user.id)">Excluir</button>
      </div>
    </li>
  </ul>
</div>

<script>
new Vue({
  el: '#app',
  data: {
    apiUrl: "http://localhost:3000/users", // JSON Server
    users: [],
    newUser: { name: "", email: "", password: "", phone: "", age: "" },
    loginEmail: "",
    loginPassword: "",
    mode: "register",
    message: "",
    messageType: "",
    isAdmin: true, // trocar para false se não quiser admin
    searchQuery: ""
  },
  created() {
    if (this.isAdmin) this.fetchUsers();
  },
  computed: {
    filteredUsers() {
      if (!this.searchQuery) return this.users;
      const q = this.searchQuery.toLowerCase();
      return this.users.filter(u =>
        u.name.toLowerCase().includes(q) || u.email.toLowerCase().includes(q)
      );
    }
  },
  methods: {
    showMessage(msg, type) {
      this.message = msg;
      this.messageType = type;
      setTimeout(() => { this.message = ""; }, 3000);
    },
    async fetchUsers() {
      try {
        const res = await axios.get(this.apiUrl);
        this.users = res.data.map(u => ({ ...u, editing: false }));
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao carregar usuários", "error");
      }
    },
    async addUser() {
      try {
        const res = await axios.post(this.apiUrl, this.newUser);
        this.users.push({ ...res.data, editing: false });
        this.showMessage("Usuário cadastrado!", "success");
        this.newUser = { name: "", email: "", password: "", phone: "", age: "" };
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao cadastrar", "error");
      }
    },
    async loginUser() {
      try {
        const res = await axios.get(`${this.apiUrl}?email=${this.loginEmail}`);
        const user = res.data.find(u => u.password === this.loginPassword);
        if (user) {
          localStorage.setItem("loggedUser", JSON.stringify(user));
          this.showMessage("Login realizado!", "success");
          setTimeout(() => { window.location.href = "perfil.html"; }, 1000);
        } else {
          this.showMessage("Email ou senha incorretos", "error");
        }
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao entrar", "error");
      }
    },
    async updateUser(user) {
      try {
        await axios.put(`${this.apiUrl}/${user.id}`, user);
        user.editing = false;
        this.showMessage("Usuário atualizado!", "success");
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao atualizar", "error");
      }
    },
    async deleteUser(id) {
      if (!confirm("Excluir este usuário?")) return;
      try {
        await axios.delete(`${this.apiUrl}/${id}`);
        this.users = this.users.filter(u => u.id !== id);
        this.showMessage("Usuário excluído!", "success");
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao excluir", "error");
      }
    },
    sortUsers() {
      this.users.sort((a, b) => a.name.localeCompare(b.name));
    }
  }
});
</script>
</body>
</html>
