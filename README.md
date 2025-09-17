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

  <!-- Alternar entre cadastro e login -->
  <div class="toggle">
    <button @click="mode = 'register'">Cadastrar</button>
    <button @click="mode = 'login'">Entrar</button>
  </div>

  <!-- Mensagem -->
  <div class="message" v-if="message" :class="messageType">{{ message }}</div>

  <!-- Formulário de cadastro -->
  <form v-if="mode === 'register'" @submit.prevent="addUser">
    <input type="text" v-model="newUser.name" placeholder="Nome" required>
    <input type="email" v-model="newUser.email" placeholder="Email" required>
    <input type="password" v-model="newUser.password" placeholder="Senha" required>
    <input type="text" v-model="newUser.phone" placeholder="Telefone">
    <input type="number" v-model="newUser.age" placeholder="Idade">
    <button type="submit">Cadastrar</button>
  </form>

  <!-- Formulário de login -->
  <form v-if="mode === 'login'" @submit.prevent="loginUser">
    <input type="email" v-model="loginEmail" placeholder="Email" required>
    <input type="password" v-model="loginPassword" placeholder="Senha" required>
    <button type="submit">Entrar</button>
  </form>

  <!-- Admin: busca e ordenação -->
  <div v-if="isAdmin">
    <input type="text" v-model="searchQuery" class="search" placeholder="Pesquisar usuário...">
    <button class="sort-btn" @click="sortUsers">Ordenar por Nome</button>
  </div>

  <!-- Lista de usuários (admin) -->
  <ul v-if="isAdmin">
    <li v-for="user in filteredUsers" :key="user.id">
      <span v-if="!user.editing">{{ user.name }} - {{ user.email }} - {{ user.phone || 'Sem telefone' }} - {{ user.age || 'Sem idade' }}</span>

      <span v-else>
        <input v-model="user.name" placeholder="Nome">
        <input v-model="user.email" placeholder="Email">
        <input v-model="user.phone" placeholder="Telefone">
        <input v-model="user.age" type="number" placeholder="Idade">
        <input v-model="user.password" type="password" placeholder="Senha">
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
    users: [],
    newUser: { name: '', email: '', password: '', phone: '', age: '' },
    loginEmail: '',
    loginPassword: '',
    mode: 'register',
    message: '',
    messageType: '',
    isAdmin: false,
    searchQuery: ''
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
    async fetchUsers() {
      try {
        const res = await axios.get('/users');
        this.users = res.data.map(u => ({ ...u, editing: false }));
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao carregar usuários!", "error");
      }
    },
    isValidEmail(email) {
      return /\S+@\S+\.\S+/.test(email);
    },
    isStrongPassword(password) {
      return /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{6,}$/.test(password);
    },
    showMessage(msg, type) {
      this.message = msg;
      this.messageType = type;
      setTimeout(() => { this.message = ''; }, 3000);
    },
    async addUser() {
      if (!this.newUser.name || !this.newUser.email || !this.newUser.password)
        return this.showMessage("Preencha os campos obrigatórios!", "error");

      const emailNorm = this.newUser.email.trim().toLowerCase();
      if (!this.isValidEmail(emailNorm))
        return this.showMessage("Digite um e-mail válido!", "error");

      if (!this.isStrongPassword(this.newUser.password))
        return this.showMessage("Senha deve ter ao menos 6 caracteres e 1 número!", "error");

      try {
        const check = await axios.get(`/users?email=${encodeURIComponent(emailNorm)}`);
        if (check.data && check.data.length) {
          return this.showMessage("E-mail já cadastrado!", "error");
        }

        const payload = {
          name: this.newUser.name.trim(),
          email: emailNorm,
          password: this.newUser.password,
          phone: this.newUser.phone ? this.newUser.phone.trim() : '',
          age: this.newUser.age || ''
        };

        const res = await axios.post('/users', payload);
        const savedUser = res.data;

        if (!this.isAdmin) {
          this.showMessage("Cadastro realizado! Redirecionando...", "success");
          setTimeout(() => {
            localStorage.setItem('loggedUser', JSON.stringify(savedUser));
            window.location.href = "perfil.html";
          }, 1500);
        } else {
          this.users.push({ savedUser, editing: false });
          this.showMessage("Usuário cadastrado com sucesso!", "success");
        }

        this.newUser = { name: '', email: '', password: '', phone: '', age: '' };
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao cadastrar usuário!", "error");
      }
    },
    async loginUser() {
      const emailNorm = this.loginEmail.trim().toLowerCase();
      const passwordInput = this.loginPassword || '';

      if (!this.isValidEmail(emailNorm)) {
        this.showMessage("Digite um e-mail válido!", "error");
        return;
      }
      if (!passwordInput) {
        this.showMessage("Digite a senha!", "error");
        return;
      }

      try {
        const res = await axios.get(`/users?email=${encodeURIComponent(emailNorm)}`);
        if (!res.data || res.data.length === 0 || res.data[0].password !== passwordInput) {
          this.showMessage("Email ou senha incorretos!", "error");
          return;
        }

        const user = res.data[0];
        localStorage.setItem('loggedUser', JSON.stringify(user));

        this.showMessage("Login realizado! Redirecionando...", "success");
        setTimeout(() => {
          window.location.href = "perfil.html";
        }, 1200);
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao entrar!", "error");
      }
    },
    async updateUser(user) {
      try {
        await axios.put(`/users/${user.id}`, {
          name: user.name,
          email: user.email,
          phone: user.phone,
          age: user.age,
          password: user.password || ''
        });
        user.editing = false;
        this.showMessage("Usuário atualizado com sucesso!", "success");
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao atualizar usuário!", "error");
      }
    },
    async deleteUser(id) {
      if (!confirm("Tem certeza que deseja excluir este usuário?")) return;
      try {
        await axios.delete(`/users/${id}`);
        this.users = this.users.filter(u => u.id !== id);
        this.showMessage("Usuário excluído com sucesso!", "success");
      } catch (err) {
        console.error(err);
        this.showMessage("Erro ao excluir usuário!", "error");
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
