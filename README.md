<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
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
    .small { font-size: 12px; color: #666; margin-bottom: 6px; text-align:center; }
    .logout { text-align:center; margin-top:10px; }
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
      <input type="text" v-model="newUser.name" placeholder="Nome" required />
      <input type="email" v-model="newUser.email" placeholder="Email" required />
      <input type="password" v-model="newUser.password" placeholder="Senha" required />
      <div class="small">Senha: mínimo 6 caracteres e pelo menos 1 número</div>
      <input type="text" v-model="newUser.phone" placeholder="Telefone" />
      <input type="number" v-model="newUser.age" placeholder="Idade" />
      <button type="submit">Cadastrar</button>
    </form>

    <!-- Formulário de login -->
    <form v-if="mode === 'login'" @submit.prevent="loginUser">
      <input type="email" v-model="loginEmail" placeholder="Email" required />
      <input type="password" v-model="loginPassword" placeholder="Senha" required />
      <button type="submit">Entrar</button>
    </form>

    <!-- Logout / informações do usuário logado -->
    <div v-if="loggedUser" class="logout">
      <div class="small">Logado como: <strong>{{ loggedUser.name }}</strong> ({{ loggedUser.email }})</div>
      <button @click="logout" style="background:#e67e22;">Sair</button>
    </div>

    <!-- Admin: busca e ordenação (apenas quando isAdmin === true) -->
    <div v-if="isAdmin" style="margin-top:20px;">
      <input type="text" v-model="searchQuery" class="search" placeholder="Pesquisar usuário..." />
      <button class="sort-btn" @click="sortUsers">Ordenar por Nome</button>
    </div>

    <!-- Lista de usuários (admin apenas) -->
    <ul v-if="isAdmin">
      <li v-for="user in filteredUsers" :key="user.id">
        <span v-if="!user.editing">{{ user.name }} - {{ user.email }} - {{ user.phone || 'Sem telefone' }} - {{ user.age || 'Sem idade' }}</span>

        <span v-else>
          <input v-model="user.name" placeholder="Nome" />
          <input v-model="user.email" placeholder="Email" />
          <input v-model="user.phone" placeholder="Telefone" />
          <input v-model="user.age" type="number" placeholder="Idade" />
          <input v-model="user.password" type="password" placeholder="Senha" />
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
        API_BASE: 'http://localhost:3000', // JSON Server base
        users: [],
        newUser: { name: '', email: '', password: '', phone: '', age: '' },
        loginEmail: '',
        loginPassword: '',
        mode: 'register', // 'register' or 'login'
        message: '',
        messageType: '',
        isAdmin: false,
        searchQuery: '',
        loggedUser: null
      },
      created() {
        // tenta recuperar usuário salvo em localStorage
        try {
          const saved = localStorage.getItem('loggedUser');
          if (saved) {
            this.loggedUser = JSON.parse(saved);
            this.isAdmin = !!this.loggedUser.isAdmin;
            if (this.isAdmin) {
              // se for admin, carrega usuários
              this.fetchUsers();
            }
            // opcional: se preferir voltar para perfil quando usuário comum estiver salvo, descomente:
            // if (!this.isAdmin) window.location.href = 'perfil.html';
          }
        } catch (e) {
          console.warn('Erro lendo localStorage:', e);
        }
      },
      computed: {
        filteredUsers() {
          if (!this.searchQuery) return this.users;
          const q = this.searchQuery.toLowerCase();
          return this.users.filter(u =>
            (u.name || '').toLowerCase().includes(q) || (u.email || '').toLowerCase().includes(q)
          );
        }
      },
      methods: {
        showMessage(msg, type) {
          this.message = msg;
          this.messageType = type;
          setTimeout(() => { this.message = ''; }, 3000);
        },
        isValidEmail(email) {
          return /\S+@\S+\.\S+/.test(email);
        },
        isStrongPassword(password) {
          return /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{6,}$/.test(password);
        },

        // lista todos usuários (admin)
        async fetchUsers() {
          try {
            const res = await axios.get(`${this.API_BASE}/users`);
            this.users = res.data.map(u => ({ ...u, editing: false }));
          } catch (err) {
            console.error('fetchUsers erro:', err);
            this.showMessage('Erro ao carregar usuários (verifique JSON Server).', 'error');
          }
        },

        // cadastrar
        async addUser() {
          if (!this.newUser.name || !this.newUser.email || !this.newUser.password) {
            return this.showMessage('Preencha os campos obrigatórios!', 'error');
          }
          const emailNorm = this.newUser.email.trim().toLowerCase();
          if (!this.isValidEmail(emailNorm)) {
            return this.showMessage('Digite um e-mail válido!', 'error');
          }
          if (!this.isStrongPassword(this.newUser.password)) {
            return this.showMessage('Senha deve ter ao menos 6 caracteres e 1 número!', 'error');
          }

          try {
            // verifica duplicata no servidor
            const check = await axios.get(`${this.API_BASE}/users?email=${encodeURIComponent(emailNorm)}`);
            if (check.data && check.data.length) {
              return this.showMessage('E-mail já cadastrado!', 'error');
            }

            const payload = {
              name: this.newUser.name.trim(),
              email: emailNorm,
              password: this.newUser.password,
              phone: this.newUser.phone ? this.newUser.phone.trim() : '',
              age: this.newUser.age || '',
              isAdmin: false
            };

            const res = await axios.post(`${this.API_BASE}/users`, payload);
            const savedUser = res.data;

            // se admin estiver logado e cadastrou manualmente, atualiza lista sem redirecionar
            if (this.isAdmin) {
              this.users.push({ ...savedUser, editing: false });
              this.showMessage('Usuário cadastrado (admin)!', 'success');
            } else {
              // salva sessão do novo usuário e redireciona para perfil
              localStorage.setItem('loggedUser', JSON.stringify(savedUser));
              this.loggedUser = savedUser;
              this.showMessage('Cadastro realizado! Redirecionando...', 'success');
              setTimeout(() => { window.location.href = 'perfil.html'; }, 1200);
            }

            this.newUser = { name: '', email: '', password: '', phone: '', age: '' };
          } catch (err) {
            console.error('addUser erro:', err);
            this.showMessage('Erro ao cadastrar usuário! (verifique JSON Server)', 'error');
          }
        },

        // login
        async loginUser() {
          const emailNorm = (this.loginEmail || '').trim().toLowerCase();
          const passwordInput = this.loginPassword || '';
          if (!this.isValidEmail(emailNorm)) {
            return this.showMessage('Digite um e-mail válido!', 'error');
          }
          if (!passwordInput) {
            return this.showMessage('Digite a senha!', 'error');
          }

          try {
            const res = await axios.get(`${this.API_BASE}/users?email=${encodeURIComponent(emailNorm)}`);
            if (!res.data || res.data.length === 0) {
              this.showMessage('Email ou senha incorretos!', 'error');
              return;
            }

            // encontro: primeiro usuário com email (deveria ser único)
            const user = res.data[0];
            if (user.password !== passwordInput) {
              this.showMessage('Email ou senha incorretos!', 'error');
              return;
            }

            // sucesso
            localStorage.setItem('loggedUser', JSON.stringify(user));
            this.loggedUser = user;
            this.isAdmin = !!user.isAdmin;
            this.showMessage('Login realizado!', 'success');

            // comportamento após login: se admin, permanece e carrega lista; se usuário comum, vai ao perfil
            setTimeout(() => {
              if (this.isAdmin) {
                this.fetchUsers();
                this.mode = 'login';
              } else {
                window.location.href = 'perfil.html';
              }
            }, 900);
          } catch (err) {
            console.error('loginUser erro:', err);
            this.showMessage('Erro ao entrar! (verifique JSON Server)', 'error');
          }
        },

        // logout
        logout() {
          localStorage.removeItem('loggedUser');
          this.loggedUser = null;
          this.isAdmin = false;
          this.showMessage('Você saiu.', 'success');
          // opcional: atualizar interface
          setTimeout(() => { this.mode = 'login'; }, 600);
        },

        // update user (admin)
        async updateUser(user) {
          try {
            const payload = {
              name: user.name,
              email: user.email,
              phone: user.phone,
              age: user.age,
              password: user.password || ''
            };
            await axios.put(`${this.API_BASE}/users/${user.id}`, payload);
            user.editing = false;
            this.showMessage('Usuário atualizado com sucesso!', 'success');

            // se admin atualizou a própria conta, atualiza localStorage
            const saved = JSON.parse(localStorage.getItem('loggedUser') || 'null');
            if (saved && saved.id === user.id) {
              localStorage.setItem('loggedUser', JSON.stringify({ ...saved, ...payload }));
              this.loggedUser = JSON.parse(localStorage.getItem('loggedUser'));
            }

            await this.fetchUsers();
          } catch (err) {
            console.error('updateUser erro:', err);
            this.showMessage('Erro ao atualizar usuário!', 'error');
          }
        },

        // delete user (admin)
        async deleteUser(id) {
          if (!confirm('Tem certeza que deseja excluir este usuário?')) return;
          try {
            await axios.delete(`${this.API_BASE}/users/${id}`);
            this.users = this.users.filter(u => u.id !== id);
            this.showMessage('Usuário excluído com sucesso!', 'success');

            // se excluiu a própria conta que está logada, desloga
            const saved = JSON.parse(localStorage.getItem('loggedUser') || 'null');
            if (saved && saved.id === id) {
              localStorage.removeItem('loggedUser');
              this.loggedUser = null;
              this.isAdmin = false;
            }
          } catch (err) {
            console.error('deleteUser erro:', err);
            this.showMessage('Erro ao excluir usuário!', 'error');
          }
        },

        sortUsers() {
          this.users.sort((a, b) => (a.name || '').localeCompare(b.name || ''));
        }
      }
    });
  </script>
</body>
</html>
