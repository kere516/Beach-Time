<template>
  <div v-if="currentUser" class="home-page">
    <header>
      <nav>
        <div class="logo">🏖️ BEACH TIME</div>
        <ul class="nav-links">
          <li><a @click="scrollToSection('home')">Início</a></li>
          <li><a @click="scrollToSection('avisos')">Avisos</a></li>
          <li><a @click="scrollToSection('quadras')">Quadras</a></li>
          <li><a @click="scrollToSection('cardapio')">Cardápio</a></li>
          <li><a @click="$router.push('/agendamento')">Agendamento</a></li>
          <li><a @click="scrollToSection('contato')">Contato</a></li>
        </ul>
        <div class="nav-actions">
          <span class="user-name">👤 {{ currentUser.name }}</span>
          <button class="btn-logout" @click="handleLogout">Deslogar</button>
        </div>
      </nav>
    </header>

    <section class="hero" id="home">
      <div class="hero-content">
        <img
          src="/images/bola.jpeg"
          alt="Beach Court"
          class="hero-image"
        >
        <div class="hero-text">
          <h1>{{ hero.title }}</h1>
          <p class="subtitle">{{ hero.subtitle }}</p>
          <p class="lorem-text">{{ hero.loremText }}</p>
        </div>
      </div>
    </section>

    <section class="avisos-section" id="avisos">
      <div class="avisos-container">
        <h2 class="section-title">📢 Avisos Importantes</h2>

        <div v-if="currentUser && currentUser.isAdmin" class="admin-panel">
          <h3>Painel Admin - Criar Aviso</h3>

          <div v-if="avisoError" class="error-message">
            {{ avisoError }}
          </div>

          <div v-if="avisoSuccess" class="success-message">
            {{ avisoSuccess }}
          </div>

          <form @submit.prevent="createAviso" class="aviso-form">
            <div class="form-group">
              <label>Título:</label>
              <input
                type="text"
                v-model="novoAviso.titulo"
                placeholder="Ex: Manutenção programada"
                required
              />
            </div>

            <div class="form-group">
              <label>Descrição:</label>
              <textarea
                v-model="novoAviso.descricao"
                placeholder="Descreva o aviso..."
                rows="4"
                required
              ></textarea>
            </div>

            <button type="submit" class="btn-criar" :disabled="isCreatingAviso">
              {{ isCreatingAviso ? 'Criando...' : '✨ Criar Aviso' }}
            </button>
          </form>
        </div>

        <div v-if="loadingAvisos" class="loading">
          Carregando avisos...
        </div>

        <div v-else-if="avisos.length === 0" class="no-avisos">
          <p>📭 Nenhum aviso no momento.</p>
        </div>

        <div v-else class="avisos-grid">
          <div
            class="aviso-card"
            v-for="aviso in avisos"
            :key="aviso.id"
          >
            <div class="aviso-header">
              <h3>{{ aviso.titulo }}</h3>
              <span class="aviso-date">{{ formatDate(aviso.createdAt) }}</span>
            </div>

            <p class="aviso-description">{{ aviso.descricao }}</p>

            <div class="aviso-footer">
              <span class="aviso-author">Por: {{ aviso.adminName }}</span>

              <button
                v-if="currentUser && currentUser.isAdmin"
                @click="deleteAviso(aviso.id)"
                class="btn-delete"
              >
                🗑️ Deletar
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section class="carousel-section" id="quadras">
      <div class="carousel-container">
        <h2 class="section-title">Nossas Quadras</h2>

        <div class="products-grid">
          <div
            class="product-card"
            v-for="(court, index) in courts"
            :key="index"
            @click="openModal(court)"
          >
            <span class="product-badge" v-if="court.discount">{{ court.discount }}</span>
            <img :src="court.image" :alt="court.title" class="product-image">
            <div class="product-info">
              <h3 class="product-title">{{ court.title }}</h3>
              <p class="product-dimensions">16 x 8 metros</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <div class="modal-overlay" v-if="selectedCourt" @click.self="closeModal">
      <div class="modal-content">
        <button class="modal-close" @click="closeModal">×</button>

        <div class="modal-images">
          <img :src="selectedCourt.image" :alt="selectedCourt.title" class="modal-main-image">
        </div>

        <div class="modal-body">
          <h2 class="modal-title">{{ selectedCourt.title }}</h2>
          <p class="modal-description">{{ selectedCourt.fullDescription }}</p>

        </div>
      </div>
    </div>

    <div class="modal-overlay" v-if="selectedMenuPhoto" @click.self="closeMenuModal">
      <div class="modal-content-menu">
        <button class="modal-close" @click="closeMenuModal">×</button>
        <img
          :src="selectedMenuPhoto.image"
          :alt="selectedMenuPhoto.name"
          class="modal-menu-image-zoom"
        >
      </div>
    </div>


    <section class="menu-section" id="cardapio">
      <div class="menu-container">
        <h2 class="section-title">Nosso Cardápio</h2>
        <p class="menu-intro">
          Aproveite o melhor da gastronomia e restabeleça a energia do seu corpo enquanto se diverte! Clique em um cardápio para ampliá-lo e visualizar todas as opções.
        </p>

        <div class="carousel-wrapper">
          <button class="carousel-btn prev" @click="prevSlide" :disabled="currentSlide === 0">
            ‹
          </button>

          <div class="menu-carousel">
            <div
              class="menu-carousel-track"
              :style="{ transform: `translateX(-${currentSlide * slideWidth}%)` }"
            >
              <div
                class="menu-card menu-photo-card"
                v-for="(item, index) in menuPhotos"
                :key="index"
                @click="openMenuModal(item)"
              >
                <img :src="item.image" :alt="item.name" class="menu-image menu-photo-image">
                <div class="menu-info">
                  <h3 class="menu-item-name">{{ item.name }}</h3>
                </div>
              </div>
            </div>
          </div>

          <button class="carousel-btn next" @click="nextSlide" :disabled="currentSlide >= maxSlide">
            ›
          </button>
        </div>

        <div class="carousel-dots">
          <span
            v-for="(dot, index) in totalDots"
            :key="index"
            class="dot"
            :class="{ active: index === currentSlide }"
            @click="goToSlide(index)"
          ></span>
        </div>
      </div>
    </section>

    <footer id="contato">
      <div class="footer-content">
        <div class="footer-section">
          <h3>Localização</h3>
          <div class="address">
            <i class="fas fa-map-marker-alt"></i>
            <p>Rua Telyus Ferraz, 160, São Benedito<br>Parnaíba - PI, 64202-470<br>Brasil</p>
          </div>
        </div>

        <div class="footer-bottom">
          <p>&copy; 2024 Beach Time. Todos os direitos reservados.</p>
          <div class="contact-links">
            <a href="https://wa.me/558695807620" target="_blank" class="whatsapp-link">
              <i class="fab fa-whatsapp"></i> Fale conosco pelo WhatsApp
            </a>
          </div>
        </div>
      </div>
    </footer>
  </div>
</template>

<script>
import dbService from '@/services/db.js'

export default {
  name: 'HomeView',
  data() {
    return {
      currentUser: null,
      selectedCourt: null,
      selectedMenuPhoto: null,
      currentSlide: 0,
      itemsPerSlide: 4,
      avisos: [],
      loadingAvisos: false,
      novoAviso: {
        titulo: '',
        descricao: ''
      },
      isCreatingAviso: false,
      avisoError: '',
      avisoSuccess: '',
      hero: {
        title: 'Quadras',
        subtitle: 'Onde o esporte encontra o estilo de vida',
        loremText: 'Cansado da rotina e em busca de um lugar perfeito para se exercitar, relaxar e dar boas risadas? Nossas quadras de areia são o cenário ideal! Aqui, a qualidade se encontra com a diversão. Projetamos nosso espaço pensando na sua performance e conforto, utilizando areia de alta qualidade, que oferece o toque e a segurança que o esporte de areia exige.'
      },
      courts: [
        {
          title: 'Quadra Crescer',
          image: '/images/quadra1.jpg',
          fullDescription: 'Quadra profissional com areia importada de alta qualidade, proporcionando a melhor experiência de jogo. Ideal para treinos e competições.',
          features: [
            'Areia importada premium',
            'Iluminação LED profissional',
            'Vestiários com chuveiro',
            'Estacionamento privativo',
            'Área de descanso coberta'
          ],
          whatsapp: '558695807620'
        },
        {
          title: 'Quadra Exata',
          image: '/images/quadra2.jpg',
          fullDescription: 'Quadra completa com boa estrutura, perfeita para jogos recreativos e treinos regulares. Excelente custo-benefício.',
          features: [
            'Areia de qualidade',
            'Arena 16x8',
            'Estacionamento disponível',
            'Bebedouro'
          ],
          whatsapp: '558695807620'
        },
        {
          title: 'Quadra Fórmula Animal',
          image: '/images/quadra3.jpg',
          fullDescription: 'Especializada em jogos noturnos com sistema de iluminação de última geração. Perfeita para quem joga após o trabalho.',
          features: [
            'Disponível até 23h',
            'Arena 16x8',
            'Vestiários completos'
          ],
          whatsapp: '558695807620'
        },
        {
          title: 'Quadra Laisa',
          image: '/images/quadra4.jpg',
          fullDescription: 'Quadra premium preparada para torneios e eventos. Conta com arquibancada, placar eletrônico e toda infraestrutura necessária.',
          features: [
            'Placar eletrônico',
            'Sistema de som'
          ],
          whatsapp: '558695807620'
        },
        {
          title: 'Quadra Tron',
          image: '/images/quadra5.jpg',
          fullDescription: 'Quadra exclusiva com acabamento premium e serviços diferenciados. Ideal para clientes que buscam o máximo em conforto e qualidade.',
          features: [
            'Areia especial importada',
            'Área VIP com ar condicionado',
            'Serviço de toalhas',
            'Bebidas inclusas',
            'Estacionamento valet'
          ],
          whatsapp: '558695807620'
        },
        {
          title: 'Quadra 60 Minutos',
          image: '/images/quadra6.jpg',
          fullDescription: 'Quadra pensada para toda a família, com área kids e espaço para confraternização. Perfeita para eventos familiares e comemorações.',
          features: [
            'Área kids segura',
            'Churrasqueira disponível',
            'Mesas para confraternização',
            'Playground próximo',
            'Banheiro família'
          ],
          whatsapp: '558695807620'
        }
      ],
      menuPhotos: [
        {
          name: 'Cardápio Cajueiro Beach House',
          image: '/images/card5.jpg'
        },
        {
          name: 'Jogada Principal',
          image: '/images/card2.jpg'
        },
        {
          name: 'Bora pro Play',
          image: '/images/card3.jpg'
        },
        {
          name: 'Aquecimento',
          image: '/images/card4.jpg'
        },
        {
          name: 'Cajueiro Grill',
          image: '/images/card1.jpg'
        }
      ]
    }
  },

  computed: {
    slideWidth() {
      return 100 / this.itemsPerSlide
    },
    maxSlide() {
      return Math.ceil(this.menuPhotos.length / this.itemsPerSlide) - 1
    },
    totalDots() {
      return this.maxSlide + 1
    }
  },

  async mounted() {
    await this.verifyAuth()
    await this.fetchAvisos()
  },

  methods: {
    async verifyAuth() {
      const token = localStorage.getItem('token')

      if (!token) {
        this.$router.push('/auth')
        return
      }

      try {
        const decoded = atob(token).split(':')
        const userId = parseInt(decoded[0])

        const user = await dbService.getUserById(userId)

        if (!user) {
          throw new Error('Usuário não encontrado')
        }

        this.currentUser = user

      } catch (error) {
        console.error('Erro na verificação:', error)
        localStorage.removeItem('token')
        localStorage.removeItem('user')
        this.$router.push('/auth')
      }
    },

    async fetchAvisos() {
      this.loadingAvisos = true

      try {
        this.avisos = await dbService.getAllAvisos()
      } catch (error) {
        console.error('Erro ao carregar avisos:', error)
      } finally {
        this.loadingAvisos = false
      }
    },

    async createAviso() {
      this.avisoError = ''
      this.avisoSuccess = ''
      this.isCreatingAviso = true

      try {
        await dbService.createAviso(
          this.novoAviso.titulo,
          this.novoAviso.descricao,
          this.currentUser.id,
          this.currentUser.name
        )

        this.avisoSuccess = 'Aviso criado com sucesso!'
        this.novoAviso.titulo = ''
        this.novoAviso.descricao = ''

        await this.fetchAvisos()

        setTimeout(() => {
          this.avisoSuccess = ''
        }, 3000)

      } catch (error) {
        this.avisoError = error.message
      } finally {
        this.isCreatingAviso = false
      }
    },

    async deleteAviso(avisoId) {
      if (!confirm('Tem certeza que deseja deletar este aviso?')) {
        return
      }

      try {
        await dbService.deleteAviso(avisoId)
        await this.fetchAvisos()
      } catch (error) {
        alert('Erro ao deletar aviso: ' + error.message)
      }
    },

    formatDate(dateString) {
      const date = new Date(dateString)
      return date.toLocaleDateString('pt-BR', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric'
      })
    },

    openModal(court) {
      this.selectedCourt = court
      document.body.style.overflow = 'hidden'
    },

    closeModal() {
      this.selectedCourt = null
      document.body.style.overflow = 'auto'
    },

    openMenuModal(photo) {
      this.selectedMenuPhoto = photo
      document.body.style.overflow = 'hidden'
    },

    closeMenuModal() {
      this.selectedMenuPhoto = null
      document.body.style.overflow = 'auto'
    },

    contactWhatsApp(court) {
      const userName = this.currentUser ? this.currentUser.name : 'Visitante'
      const message = `Olá! Meu nome é ${userName} e gostaria de saber sobre valores e formas de pagamento para a ${court.title}.`
      const url = `https://wa.me/558695807620?text=${encodeURIComponent(message)}`
      window.open(url, '_blank')
    },

    scrollToSection(sectionId) {
      const element = document.getElementById(sectionId)
      if (element) {
        element.scrollIntoView({ behavior: 'smooth' })
      }
    },

    handleLogout() {
      if (confirm('Deseja realmente deslogar?')) {
        localStorage.removeItem('token')
        localStorage.removeItem('user')
        this.$router.push('/auth')
      }
    },

    nextSlide() {
      if (this.currentSlide < this.maxSlide) {
        this.currentSlide++
      }
    },

    prevSlide() {
      if (this.currentSlide > 0) {
        this.currentSlide--
      }
    },

    goToSlide(index) {
      this.currentSlide = index
    }
  }
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.home-page {
  width: 100%;
  min-height: 100vh;
}

header {
  background: linear-gradient(135deg, #000000 0%, #1a1a1a 100%);
  color: white;
  padding: 1.5rem 0;
  position: sticky;
  top: 0;
  z-index: 1000;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

nav {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  font-size: 1.8rem;
  font-weight: bold;
  letter-spacing: 1px;
  background: linear-gradient(135deg, #ff9933 0%, #ffb366 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.nav-links {
  display: flex;
  gap: 2.5rem;
  list-style: none;
}

.nav-links a {
  color: #b8b8b8;
  text-decoration: none;
  font-size: 0.95rem;
  transition: all 0.3s;
  cursor: pointer;
}

.nav-links a:hover {
  color: #94e432;
}

.nav-actions {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.user-name {
  color: #94e432;
  font-weight: 600;
  font-size: 0.95rem;
}

.btn-logout {
  background: transparent;
  color: #ff6b6b;
  border: 1px solid #ff6b6b;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.85rem;
  transition: all 0.3s;
}

.btn-logout:hover {
  background: #ff6b6b;
  color: white;
}

.hero {
  max-width: 1400px;
  margin: 4rem auto;
  padding: 4rem 2rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
}

.hero-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 4rem;
  align-items: center;
}

.hero-image {
  width: 100%;
  height: 450px;
  object-fit: cover;
  border-radius: 8px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
}

h1 {
  font-size: 3.5rem;
  margin-bottom: 1.5rem;
  color: #1a1a1a;
  font-weight: 700;
  text-align: center;
}

.subtitle {
  font-size: 1.3rem;
  color: #94e432;
  margin-bottom: 2rem;
  font-weight: 600;
  text-align: center;
}

.lorem-text {
  font-size: 1.05rem;
  line-height: 1.8;
  color: #666;
  text-align: justify;
}

.avisos-section {
  padding: 4rem 0;
  margin-top: 3rem;
  background: #fff3e0;
}

.avisos-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem;
}

.admin-panel {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08);
  margin-bottom: 3rem;
  border-left: 4px solid #ff9933;
}

.admin-panel h3 {
  color: #1a1a1a;
  margin-bottom: 1.5rem;
  font-size: 1.3rem;
}

.aviso-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-group label {
  font-weight: 600;
  color: #1a1a1a;
  font-size: 0.95rem;
}

.form-group input,
.form-group textarea {
  padding: 0.8rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  font-family: inherit;
  transition: border-color 0.3s;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #94e432;
}

.btn-criar {
  background: linear-gradient(to right, #ff9933, #ff7700);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  align-self: flex-start;
}

.btn-criar:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(255, 153, 51, 0.3);
}

.btn-criar:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.loading {
  text-align: center;
  padding: 3rem;
  color: #666;
  font-size: 1.1rem;
}

.no-avisos {
  text-align: center;
  padding: 3rem;
  background: white;
  border-radius: 12px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08);
}

.no-avisos p {
  color: #999;
  font-size: 1.2rem;
}

.avisos-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 2rem;
}

.aviso-card {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08);
  border-left: 4px solid #ff9933;
  transition: transform 0.3s, box-shadow 0.3s;
}

.aviso-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(255, 153, 51, 0.2);
}

.aviso-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
  gap: 1rem;
}

.aviso-header h3 {
  color: #1a1a1a;
  font-size: 1.3rem;
  font-weight: 700;
  flex: 1;
}

.aviso-date {
  background: #fff3e0;
  color: #94e432;
  padding: 0.3rem 0.8rem;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 600;
  white-space: nowrap;
}

.aviso-description {
  color: #666;
  line-height: 1.6;
  margin-bottom: 1.5rem;
  font-size: 1rem;
}

.aviso-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1rem;
  border-top: 1px solid #f0f0f0;
}

.aviso-author {
  color: #999;
  font-size: 0.9rem;
  font-style: italic;
}

.btn-delete {
  background: transparent;
  color: #ff3333;
  border: 1px solid #ff3333;
  padding: 0.4rem 1rem;
  border-radius: 6px;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-delete:hover {
  background: #ff3333;
  color: white;
}

.error-message {
  background: #ffebee;
  color: #c62828;
  padding: 1rem;
  border-radius: 8px;
  border-left: 4px solid #c62828;
  margin-bottom: 1rem;
  font-size: 0.95rem;
}

.success-message {
  background: #000000;
  color: #94e432;
  padding: 1rem;
  border-radius: 8px;
  border-left: 4px solid #94e432;
  margin-bottom: 1rem;
  font-size: 0.95rem;
}

.carousel-section {
  padding: 4rem 0;
  margin-top: 3rem;
}

.carousel-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem;
}

.section-title {
  font-size: 2.5rem;
  text-align: center;
  margin-bottom: 3rem;
  color: #94e432;
  font-weight: 700;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 2rem;
}

.product-card {
  background: #000000;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08);
  transition: transform 0.3s, box-shadow 0.3s;
  cursor: pointer;
  position: relative;
}

.product-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 15px 40px rgba(148, 228, 50, 0.2);
}

.product-image {
  width: 100%;
  height: 250px;
  object-fit: cover;
}

.product-info {
  padding: 1.5rem;
}

.product-title {
  font-size: 1.2rem;
  font-weight: 700;
  color: #94e432;
  margin-bottom: 0.8rem;
}

.product-dimensions {
  font-size: 0.95rem;
  color: #94e432;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: 2rem;
}

.modal-content {
  background: #000000;
  border-radius: 12px;
  max-width: 900px;
  width: 100%;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
}

.modal-content-menu {
  background: #000000;
  border-radius: 12px;
  max-width: 95%;
  width: auto;
  max-height: 95vh;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-menu-image-zoom {
  max-width: 100%;
  max-height: 90vh;
  object-fit: contain;
  display: block;
  border-radius: 12px;
}

.modal-close {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: #94e432;
  color: #000000;
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  font-size: 1.5rem;
  cursor: pointer;
  z-index: 10;
  opacity: 0.8;
  transition: opacity 0.3s;
}

.modal-close:hover {
  opacity: 1;
}

.modal-images {
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
  padding: 1rem 0;
}

.modal-main-image {
  max-width: 100%;
  max-height: 60vh;
  width: auto;
  height: auto;
  object-fit: contain;
  object-fit: cover;
  border-radius: 8px;
}

.modal-body {
  padding: 0 2rem 2rem;
}

.modal-title {
  font-size: 2rem;
  font-weight: 700;
  color: #94e432;
  margin-bottom: 1rem;
}

.modal-description {
  font-size: 1.05rem;
  line-height: 1.8;
  color: #94e432;
  margin-bottom: 2rem;
}

.modal-features {
  background: #000000;
  padding: 1.5rem;
  border-radius: 8px;
  margin-bottom: 2rem;
}

.modal-features h3 {
  font-size: 1.3rem;
  color: #94e432;
  margin-bottom: 1rem;
}

.modal-features ul {
  list-style: none;
}

.modal-features li {
  padding: 0.5rem 0;
  color: #94e432;
}

.modal-features li:before {
  content: "✓ ";
  color: #94e432;
  font-weight: bold;
  margin-right: 0.5rem;
}

.modal-payment {
  text-align: center;
  padding: 2rem;
  background: #000000;
  border-radius: 8px;
}

.modal-payment h3 {
  font-size: 1.5rem;
  color: #94e432;
  margin-bottom: 1rem;
}

.whatsapp-btn {
  background: #94e432;
  color: #000000;
  padding: 1rem 3rem;
  border: none;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 4px 15px rgba(148, 228, 50, 0.3);
  display: inline-flex;
  align-items: center;
  gap: 0.8rem;
}

.whatsapp-btn:hover {
  transform: translateY(-3px);
  background: #7bc02a;
}

.menu-section {
  padding: 3rem 0;
  margin-top: 3rem;
  background: #000000;
}

.menu-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 2rem;
}

.menu-intro {
  text-align: center;
  max-width: 700px;
  margin: 0 auto 2.5rem;
  font-size: 1rem;
  line-height: 1.6;
  color: #666;
}

.carousel-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.carousel-btn {
  background: #94e432;
  color: white;
  border: none;
  width: 45px;
  height: 45px;
  border-radius: 50%;
  font-size: 1.8rem;
  cursor: pointer;
  transition: all 0.3s;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 10px rgba(255, 153, 51, 0.3);
}

.carousel-btn:hover:not(:disabled) {
  background: #7bc02a;
  transform: scale(1.1);
}

.carousel-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
  opacity: 0.5;
}

.menu-carousel {
  flex: 1;
  overflow: hidden;
  background: white;
  padding: 2.5rem;
  border-radius: 12px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08);
}

.menu-carousel-track {
  display: flex;
  transition: transform 0.4s ease-in-out;
}

.menu-card {
  min-width: 25%;
  background: white;
  border: 2px solid #f0f0f0;
  border-radius: 8px;
  padding: 1.5rem;
  margin: 0 0.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  transition: all 0.3s;
  cursor: pointer;
  box-sizing: border-box;
}

.menu-card:hover {
  border-color: #94e432;
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(255, 153, 51, 0.15);
}

.menu-photo-card {
  padding: 1rem;
}

.menu-image {
  width: 100%;
  height: 250px;
  object-fit: cover;
  margin-bottom: 1rem;
  border-radius: 8px;
}

.menu-photo-image {
  height: 200px;
  object-fit: cover;
}

.menu-info {
  text-align: center;
  width: 100%;
}

.menu-item-name {
  font-size: 1.1rem;
  font-weight: 600;
  color: #1a1a1a;
  margin-bottom: 0;
}

.carousel-dots {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #ddd;
  cursor: pointer;
  transition: all 0.3s;
}

.dot.active {
  background: #94e432;
  width: 25px;
  border-radius: 5px;
}

.dot:hover {
  background: #a9e85e;
}

footer {
  background: linear-gradient(135deg, #000000 0%, #1a1a1a 100%);
  color: white;
  padding: 4rem 2rem 2rem;
  margin-top: 5rem;
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  text-align: center;
}

.footer-section {
  margin-bottom: 2.5rem;
}

.footer-section h3 {
  color: #94e432;
  font-size: 1.5rem;
  margin-bottom: 1.5rem;
  position: relative;
  display: inline-block;
  padding-bottom: 0.5rem;
}

.footer-section h3:after {
  content: '';
  position: absolute;
  width: 50px;
  height: 2px;
  background: #94e432;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
}

.address {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  margin-bottom: 2rem;
}

.address i {
  font-size: 2rem;
  color: #94e432;
}

.address p {
  color: #b8b8b8;
  line-height: 1.6;
  margin: 0;
}

.footer-bottom {
  padding-top: 2rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.footer-bottom p {
  color: #b8b8b8;
  margin-bottom: 1.5rem;
}

.contact-links {
  margin-top: 1.5rem;
}

.whatsapp-link {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  background: #94e432;
  color: white;
  padding: 0.8rem 1.5rem;
  border-radius: 50px;
  text-decoration: none;
  font-weight: 500;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(148, 228, 50, 0.3);
}

.whatsapp-link:hover {
  background: #7bc02a;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(148, 228, 50, 0.4);
}

.whatsapp-link i {
  font-size: 1.2rem;
}

@media (max-width: 1200px) {
  .products-grid {
    grid-template-columns: repeat(3, 1fr);
  }

  .menu-card {
    min-width: 33.333%;
  }
}

@media (max-width: 768px) {
  h1 {
    font-size: 2.5rem;
  }

  .nav-links {
    display: none;
  }

  .hero-content {
    grid-template-columns: 1fr;
  }

  .hero-image {
    height: 300px;
  }

  .products-grid {
    grid-template-columns: 1fr;
  }

  .avisos-grid {
    grid-template-columns: 1fr;
  }

  .aviso-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .aviso-footer {
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }

  .btn-delete {
    width: 100%;
  }

  .carousel-wrapper {
    flex-direction: column;
    gap: 0.5rem;
  }

  .carousel-btn {
    width: 40px;
    height: 40px;
    font-size: 1.5rem;
  }

  .carousel-btn.prev,
  .carousel-btn.next {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    z-index: 10;
  }

  .carousel-btn.prev {
    left: -10px;
  }

  .carousel-btn.next {
    right: -10px;
  }

  .menu-carousel {
    width: 100%;
    padding: 1.5rem 1rem;
  }

  .menu-card {
    min-width: 100%;
    margin: 0 0.25rem;
  }

  .menu-intro {
    font-size: 0.95rem;
    padding: 0 1rem;
    margin-bottom: 2rem;
  }

  .modal-images {
    grid-template-columns: 1fr;
  }

  .modal-content-menu {
    max-width: 100%;
    max-height: 100vh;
  }
}
</style>
