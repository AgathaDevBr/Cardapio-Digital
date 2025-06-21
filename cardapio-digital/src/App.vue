<template>
  <div class="container">
    <!-- Header -->
    <header class="header">
      <div class="header-content">
        <h1>🍔 Delícias Express</h1>
        <p>Sabor que chega até você!</p>
      </div>
    </header>

    <!-- Menu horizontal -->
    <nav class="menu">
      <div class="menu-tabs">
        <button
          v-for="(category, index) in menu"
          :key="index"
          @click="activeTab = category.name"
          :class="{ active: activeTab === category.name }"
          class="tab-button"
        >
          <span class="tab-icon">{{ category.icon }}</span>
          <span class="tab-text">{{ category.name }}</span>
        </button>
      </div>
    </nav>

    <!-- Conteúdo -->
    <div class="tab-content">
      <div class="category-title">
        <h2>{{ activeTab }}</h2>
      </div>
      <div class="items-grid">
        <div v-for="item in currentItems" :key="item.name" class="card">
          <div class="card-image">
            <img :src="item.image" :alt="item.name" />
            <div class="card-badge" v-if="item.popular">🔥 Popular</div>
          </div>
          <div class="card-content">
            <h3>{{ item.name }}</h3>
            <p class="description">{{ item.description }}</p>
            <div class="price-section">
              <span class="price">R$ {{ item.price.toFixed(2).replace('.', ',') }}</span>
              <div class="quantity-control">
                <button @click="decreaseQuantity(item)" class="qty-btn">-</button>
                <input
                  type="number"
                  v-model.number="item.quantity"
                  min="0"
                  class="qty-input"
                  @input="calcularTotal"
                />
                <button @click="increaseQuantity(item)" class="qty-btn">+</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Carrinho fixo -->
    <div class="cart-summary" v-if="totalItems > 0">
      <div class="cart-info">
        <span class="cart-items">{{ totalItems }} {{ totalItems === 1 ? 'item' : 'itens' }}</span>
        <span class="cart-total">R$ {{ total.toFixed(2).replace('.', ',') }}</span>
      </div>
      <button @click="showCheckout = true" class="checkout-btn">
        Finalizar Pedido
      </button>
    </div>

    <!-- Modal de Checkout -->
    <div v-if="showCheckout" class="modal-overlay" @click="closeModal">
      <div class="modal" @click.stop>
        <div class="modal-header">
          <h2>🛍️ Finalizar Pedido</h2>
          <button @click="showCheckout = false" class="close-btn">&times;</button>
        </div>
        
        <div class="modal-content">
          <!-- Resumo do pedido -->
          <div class="order-summary">
            <h3>Seu Pedido:</h3>
            <div v-for="item in selectedItems" :key="item.name" class="order-item">
              <span>{{ item.quantity }}x {{ item.name }}</span>
              <span>R$ {{ (item.price * item.quantity).toFixed(2).replace('.', ',') }}</span>
            </div>
            <div class="order-total">
              <strong>Total: R$ {{ total.toFixed(2).replace('.', ',') }}</strong>
            </div>
          </div>

          <!-- Formulário de endereço -->
          <div class="address-form">
            <h3>📍 Endereço de Entrega:</h3>
          <div class="address-form">
            <h3>📍 Endereço de Entrega:</h3>
            <div class="form-group">
              <label>Nome Completo:</label>
              <input v-model="customerInfo.name" type="text" placeholder="Seu nome completo" required>
            </div>
            <div class="form-group">
              <label>Telefone:</label>
              <input v-model="customerInfo.phone" type="tel" placeholder="(00) 00000-0000" required>
            </div>
            <div class="form-group">
              <label>CEP:</label>
              <input v-model="customerInfo.cep" type="text" placeholder="00000-000" required>
            </div>
            <div class="form-group">
              <label>Rua:</label>
              <input v-model="customerInfo.street" type="text" placeholder="Nome da rua" required>
            </div>
            <div class="form-row">
              <div class="form-group">
                <label>Número:</label>
                <input v-model="customerInfo.number" type="text" placeholder="123" required>
              </div>
              <div class="form-group">
                <label>Complemento:</label>
                <input v-model="customerInfo.complement" type="text" placeholder="Apto, bloco...">
              </div>
            </div>
            <div class="form-group">
              <label>Bairro:</label>
              <input v-model="customerInfo.neighborhood" type="text" placeholder="Nome do bairro" required>
            </div>
            <div class="form-group">
              <label>Cidade:</label>
              <input v-model="customerInfo.city" type="text" placeholder="Nome da cidade" required>
            </div>
            <div class="form-group">
              <label>Observações:</label>
              <textarea v-model="customerInfo.observations" placeholder="Alguma observação especial?"></textarea>
            </div>
          </div>

          <!-- Forma de pagamento -->
          <div class="payment-method">
            <h3>💳 Forma de Pagamento:</h3>
            <div class="payment-options">
              <label class="payment-option">
                <input type="radio" v-model="customerInfo.payment" value="dinheiro">
                <span>💵 Dinheiro</span>
              </label>
              <label class="payment-option">
                <input type="radio" v-model="customerInfo.payment" value="cartao">
                <span>💳 Cartão</span>
              </label>
              <label class="payment-option">
                <input type="radio" v-model="customerInfo.payment" value="pix">
                <span>📱 PIX</span>
              </label>
            </div>
            <div v-if="customerInfo.payment === 'dinheiro'" class="change-input">
              <label>Troco para quanto?</label>
              <input v-model="customerInfo.change" type="number" placeholder="0,00">
            </div>
          </div>
        </div>

        <div class="modal-footer">
          <button @click="sendWhatsAppOrder" class="whatsapp-btn" :disabled="!isFormValid">
            <span>📱</span>
            Enviar Pedido via WhatsApp
          </button>
        </div>
      </div>
    </div>
   </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed } from 'vue';

const menu = reactive([
  {
    name: 'Lanches',
    icon: '🍔',
    items: [
      {
        name: 'X-Burger Clássico',
        description: 'Pão brioche, hambúrguer 150g, queijo, alface, tomate',
        price: 18.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1568901346375-23c9450c58cd?w=300&h=200&fit=crop',
        popular: true
      },
      {
        name: 'X-Bacon Especial',
        description: 'Hambúrguer 180g, bacon crocante, queijo cheddar, cebola caramelizada',
        price: 24.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1553979459-d2229ba7433a?w=300&h=200&fit=crop'
      },
      {
        name: 'X-Salada Fitness',
        description: 'Hambúrguer de frango, queijo branco, alface, tomate, pepino',
        price: 21.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1572802419224-296b0aeee0d9?w=300&h=200&fit=crop'
      },
      {
        name: 'X-Tudo Supremo',
        description: 'Duplo hambúrguer, bacon, ovo, queijo, salada completa',
        price: 32.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1594212699903-ec8a3eca50f5?w=300&h=200&fit=crop',
        popular: true
      },
      {
        name: 'X-Vegano',
        description: 'Hambúrguer de grão-de-bico, queijo vegano, rúcula, tomate seco',
        price: 26.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1525059696034-4967a729002e?w=300&h=200&fit=crop'
      },
      {
        name: 'X-Frango Crispy',
        description: 'Frango empanado crocante, maionese especial, alface americana',
        price: 22.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1606755962773-d324e9a13086?w=300&h=200&fit=crop'
      }
    ]
  },
  {
    name: 'Porções',
    icon: '🍟',
    items: [
      {
        name: 'Batata Frita Tradicional',
        description: 'Batatas cortadas na hora, temperadas com sal especial',
        price: 16.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1573080496219-bb080dd4f877?w=300&h=200&fit=crop',
        popular: true
      },
      {
        name: 'Batata com Cheddar e Bacon',
        description: 'Batatas fritas cobertas com cheddar cremoso e bacon',
        price: 24.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1630384060421-cb20d0e0649d?w=300&h=200&fit=crop'
      },
      {
        name: 'Frango à Passarinho',
        description: 'Pedaços de frango temperados e fritos, acompanha molho especial',
        price: 28.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1562967914-608f82629710?w=300&h=200&fit=crop'
      },
      {
        name: 'Onion Rings',
        description: 'Anéis de cebola empanados e dourados, servidos com molho ranch',
        price: 19.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1639024471283-03518883512d?w=300&h=200&fit=crop'
      },
      {
        name: 'Mandioca Frita',
        description: 'Mandioca cortada em bastões, frita e temperada',
        price: 14.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1608039829572-78524f79c4c7?w=300&h=200&fit=crop'
      },
      {
        name: 'Mix de Petiscos',
        description: 'Batata, onion rings, nuggets e molhos variados',
        price: 35.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1541592106381-b31e9677c0e5?w=300&h=200&fit=crop'
      }
    ]
  },
  {
    name: 'Bebidas',
    icon: '🥤',
    items: [
      {
        name: 'Refrigerante Lata',
        description: 'Coca-Cola, Pepsi, Guaraná, Fanta - 350ml',
        price: 5.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1581636625402-29b2a704ef13?w=300&h=200&fit=crop'
      },
      {
        name: 'Refrigerante 2L',
        description: 'Coca-Cola, Pepsi, Guaraná, Fanta - 2 litros',
        price: 12.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1629203851122-3726ecdf080e?w=300&h=200&fit=crop'
      },
      {
        name: 'Suco Natural 500ml',
        description: 'Laranja, limão, maracujá, acerola - feito na hora',
        price: 8.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1600271886742-f049cd451bba?w=300&h=200&fit=crop',
        popular: true
      },
      {
        name: 'Água Mineral',
        description: 'Água mineral sem gás - 500ml',
        price: 3.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1559839734-2b71ea197ec2?w=300&h=200&fit=crop'
      },
      {
        name: 'Água com Gás',
        description: 'Água mineral com gás - 500ml',
        price: 4.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1523362628745-0c100150b504?w=300&h=200&fit=crop'
      },
      {
        name: 'Cerveja Long Neck',
        description: 'Cerveja gelada - 355ml (apenas para maiores de 18 anos)',
        price: 7.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1608270586620-248524c67de9?w=300&h=200&fit=crop'
      },
      {
        name: 'Milkshake',
        description: 'Chocolate, morango ou baunilha - 400ml',
        price: 14.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1572490122747-3968b75cc699?w=300&h=200&fit=crop'
      },
      {
        name: 'Energético',
        description: 'Red Bull, Monster - 250ml',
        price: 9.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1622543925917-763c34d1a86e?w=300&h=200&fit=crop'
      }
    ]
  },
  {
    name: 'Sobremesas',
    icon: '🍰',
    items: [
      {
        name: 'Brownie com Sorvete',
        description: 'Brownie de chocolate quente com bola de sorvete de baunilha',
        price: 16.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1606313564200-e75d5e30476c?w=300&h=200&fit=crop',
        popular: true
      },
      {
        name: 'Torta de Limão',
        description: 'Fatia generosa de torta de limão com merengue',
        price: 12.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1565958011703-44f9829ba187?w=300&h=200&fit=crop'
      },
      {
        name: 'Pudim Caseiro',
        description: 'Pudim de leite condensado com calda de caramelo',
        price: 10.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1551024506-0bccd828d307?w=300&h=200&fit=crop'
      },
      {
        name: 'Açaí 500ml',
        description: 'Açaí cremoso com granola, banana e mel',
        price: 18.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1571115764595-644a1f56a55c?w=300&h=200&fit=crop'
      },
      {
        name: 'Sorvete 2 Bolas',
        description: 'Chocolate, morango, baunilha ou flocos',
        price: 8.90,
        quantity: 0,
        image: 'https://images.unsplash.com/photo-1563805042-7684c019e1cb?w=300&h=200&fit=crop'
      }
    ]
  }
]);

const activeTab = ref('Lanches');
const total = ref(0);
const showCheckout = ref(false);

const customerInfo = reactive({
  name: '',
  phone: '',
  cep: '',
  street: '',
  number: '',
  complement: '',
  neighborhood: '',
  city: '',
  observations: '',
  payment: 'dinheiro',
  change: ''
});

const currentItems = computed(() => {
  return menu.find(category => category.name === activeTab.value).items;
});

const selectedItems = computed(() => {
  const items = [];
  menu.forEach(category => {
    category.items.forEach(item => {
      if (item.quantity > 0) {
        items.push(item);
      }
    });
  });
  return items;
});

const totalItems = computed(() => {
  let count = 0;
  menu.forEach(category => {
    category.items.forEach(item => {
      count += item.quantity || 0;
    });
  });
  return count;
});

const isFormValid = computed(() => {
  return customerInfo.name &&
         customerInfo.phone &&
         customerInfo.street &&
         customerInfo.number &&
         customerInfo.neighborhood &&
         customerInfo.city &&
         customerInfo.payment;
});

function increaseQuantity(item) {
  item.quantity = (item.quantity || 0) + 1;
  calcularTotal();
}

function decreaseQuantity(item) {
  if (item.quantity > 0) {
    item.quantity--;
    calcularTotal();
  }
}

function calcularTotal() {
  total.value = 0;
  menu.forEach(category => {
    category.items.forEach(item => {
      total.value += item.price * (item.quantity || 0);
    });
  });
}

function closeModal(event) {
  if (event.target === event.currentTarget) {
    showCheckout.value = false;
  }
}

function sendWhatsAppOrder() {
  if (!isFormValid.value) {
    alert('Por favor, preencha todos os campos obrigatórios!');
    return;
  }

  const phone = '21983309387'; // Substitua pelo seu número
  let message = `🍔 *NOVO PEDIDO - Delícias Express*%0A%0A`;
  
  // Dados do cliente
  message += `👤 *Cliente:* ${customerInfo.name}%0A`;
  message += `📱 *Telefone:* ${customerInfo.phone}%0A%0A`;
  
  // Endereço
  message += `📍 *Endereço de Entrega:*%0A`;
  message += `${customerInfo.street}, ${customerInfo.number}`;
  if (customerInfo.complement) {
    message += ` - ${customerInfo.complement}`;
  }
  message += `%0A${customerInfo.neighborhood} - ${customerInfo.city}%0A`;
  message += `CEP: ${customerInfo.cep}%0A%0A`;
  
  // Itens do pedido
  message += `🛍️ *Itens do Pedido:*%0A`;
  selectedItems.value.forEach(item => {
    message += `• ${item.quantity}x ${item.name}%0A`;
    message += `  R$ ${(item.price * item.quantity).toFixed(2).replace('.', ',')}%0A`;
  });
  
  // Total
  message += `%0A💰 *Total: R$ ${total.value.toFixed(2).replace('.', ',')}*%0A%0A`;
  
  // Pagamento
  message += `💳 *Forma de Pagamento:* ${customerInfo.payment.toUpperCase()}`;
  if (customerInfo.payment === 'dinheiro' && customerInfo.change) {
    message += ` (Troco para R$ ${customerInfo.change})`;
  }
  message += `%0A`;
  
  // Observações
  if (customerInfo.observations) {
    message += `%0A📝 *Observações:* ${customerInfo.observations}`;
  }

  const whatsappUrl = `https://wa.me/${phone}?text=${message}`;
  window.open(whatsappUrl, '_blank');
  
  // Reset form
  showCheckout.value = false;
  resetOrder();
}

function resetOrder() {
  menu.forEach(category => {
    category.items.forEach(item => {
      item.quantity = 0;
    });
  });
  total.value = 0;
  Object.keys(customerInfo).forEach(key => {
    if (key === 'payment') {
      customerInfo[key] = 'dinheiro';
    } else {
      customerInfo[key] = '';
    }
  });
}
</script>

<style scoped>
/* Reset e configurações globais */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  height: 100%;
  width: 100%;
  overflow-x: hidden;
}

.container {
  width: 100vw;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  position: relative;
  padding-bottom: 120px;
}

/* Header */
.header {
  background: linear-gradient(135deg, #ff6b6b, #ee5a24);
  color: white;
  text-align: center;
  padding: 2rem 1rem;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
  width: 100%;
}

.header-content h1 {
  margin: 0;
  font-size: clamp(1.8rem, 4vw, 2.5rem);
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.header-content p {
  margin: 0.5rem 0 0;
  font-size: clamp(1rem, 2.5vw, 1.2rem);
  opacity: 0.9;
}

/* Menu Navigation */
.menu {
  background: white;
  padding: 1rem;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  position: sticky;
  top: 0;
  z-index: 100;
  width: 100%;
}

.menu-tabs {
  display: flex;
  gap: 0.5rem;
  overflow-x: auto;
  padding-bottom: 0.5rem;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.menu-tabs::-webkit-scrollbar {
  display: none;
}

.tab-button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1.5rem;
  border: none;
  background: #f8f9fa;
  border-radius: 25px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  white-space: nowrap;
  min-width: fit-content;
  flex-shrink: 0;
}

.tab-button:hover {
  background: #e9ecef;
  transform: translateY(-2px);
}

.tab-button.active {
  background: linear-gradient(135deg, #ff6b6b, #ee5a24);
  color: white;
  box-shadow: 0 4px 15px rgba(255, 107, 107, 0.4);
}

.tab-icon {
  font-size: 1.2rem;
}

.tab-text {
  font-size: clamp(0.8rem, 2vw, 1rem);
}

/* Content */
.tab-content {
  padding: 2rem 1rem;
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
}

.category-title {
  text-align: center;
  margin-bottom: 2rem;
}

.category-title h2 {
  color: white;
  font-size: clamp(1.5rem, 4vw, 2rem);
  margin: 0;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.items-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
  width: 100%;
}

/* Cards */
.card {
  background: white;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 8px 30px rgba(0,0,0,0.12);
  transition: all 0.3s ease;
  position: relative;
  width: 100%;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 15px 40px rgba(0,0,0,0.2);
}

.card-image {
  position: relative;
  height: 200px;
  overflow: hidden;
  width: 100%;
}

.card-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.card:hover .card-image img {
  transform: scale(1.05);
}

.card-badge {
  position: absolute;
  top: 10px;
  right: 10px;
  background: linear-gradient(135deg, #ff6b6b, #ee5a24);
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: bold;
}

.card-content {
  padding: 1.5rem;
}

.card-content h3 {
  margin: 0 0 0.5rem;
  font-size: clamp(1.1rem, 2.5vw, 1.3rem);
  color: #2c3e50;
}

.description {
  color: #7f8c8d;
  font-size: clamp(0.8rem, 2vw, 0.9rem);
  margin: 0 0 1rem;
  line-height: 1.4;
}

.price-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.price {
  font-size: clamp(1.2rem, 3vw, 1.4rem);
  font-weight: bold;
  color: #27ae60;
}

.quantity-control {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.qty-btn {
  width: 35px;
  height: 35px;
  border: none;
  border-radius: 50%;
  background: linear-gradient(135deg, #ff6b6b, #ee5a24);
  color: white;
  font-size: 1.2rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.qty-btn:hover {
  transform: scale(1.1);
}

.qty-input {
  width: 50px;
  height: 35px;
  text-align: center;
  border: 2px solid #e9ecef;
  border-radius: 8px;
  font-weight: bold;
  font-size: 1rem;
}

/* Cart Summary */
.cart-summary {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: white;
  padding: 1rem;
  box-shadow: 0 -4px 20px rgba(0,0,0,0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
  z-index: 1000;
  width: 100vw;
}

.cart-info {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.cart-items {
  font-size: clamp(0.8rem, 2vw, 0.9rem);
  color: #7f8c8d;
}

.cart-total {
  font-size: clamp(1.1rem, 3vw, 1.3rem);
  font-weight: bold;
  color: #27ae60;
}

.checkout-btn {
  background: linear-gradient(135deg, #00b894, #00a085);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 25px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: clamp(0.9rem, 2vw, 1rem);
}

.checkout-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 184, 148, 0.4);
}

/* Modal */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
  padding: 1rem;
  width: 100vw;
  height: 100vh;
}

.modal {
  background: white;
  border-radius: 20px;
  max-width: 600px;
  width: 100%;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0,0,0,0.3);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid #e9ecef;
  background: linear-gradient(135deg, #ff6b6b, #ee5a24);
  color: white;
  border-radius: 20px 20px 0 0;
}

.modal-header h2 {
  margin: 0;
  font-size: clamp(1.2rem, 3vw, 1.5rem);
}

.close-btn {
  background: none;
  border: none;
  font-size: 2rem;
  color: white;
  cursor: pointer;
  padding: 0;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background 0.2s ease;
}

.close-btn:hover {
  background: rgba(255,255,255,0.2);
}

.modal-content {
  padding: 1.5rem;
}

/* Order Summary */
.order-summary {
  margin-bottom: 2rem;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 10px;
}

.order-summary h3 {
  margin: 0 0 1rem;
  color: #2c3e50;
  font-size: clamp(1rem, 2.5vw, 1.2rem);
}

.order-item {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem 0;
  border-bottom: 1px solid #e9ecef;
  font-size: clamp(0.8rem, 2vw, 0.9rem);
}

.order-item:last-child {
  border-bottom: none;
}

.order-total {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 2px solid #dee2e6;
  display: flex;
  justify-content: space-between;
  font-weight: bold;
  font-size: clamp(1.1rem, 3vw, 1.3rem);
  color: #27ae60;
}

/* Form Styles */
.address-form, .payment-method {
  margin-bottom: 2rem;
}

.address-form h3, .payment-method h3 {
  margin: 0 0 1rem;
  color: #2c3e50;
  font-size: clamp(1rem, 2.5vw, 1.2rem);
}

.form-group {
  margin-bottom: 1rem;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #2c3e50;
  font-size: clamp(0.8rem, 2vw, 0.9rem);
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e9ecef;
  border-radius: 8px;
  font-size: clamp(0.8rem, 2vw, 1rem);
  transition: border-color 0.3s ease;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #ff6b6b;
  box-shadow: 0 0 0 3px rgba(255, 107, 107, 0.1);
}

.form-group textarea {
  resize: vertical;
  min-height: 80px;
}

/* Payment Options */
.payment-options {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.payment-option {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem;
  border: 2px solid #e9ecef;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: clamp(0.9rem, 2vw, 1rem);
}

.payment-option:hover {
  border-color: #ff6b6b;
  background: rgba(255, 107, 107, 0.05);
}

.payment-option input[type="radio"] {
  width: auto;
  margin: 0;
}

.payment-option input[type="radio"]:checked + span {
  color: #ff6b6b;
  font-weight: bold;
}

.change-input {
  margin-top: 1rem;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 8px;
}

.change-input label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #2c3e50;
  font-size: clamp(0.8rem, 2vw, 0.9rem);
}

.change-input input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e9ecef;
  border-radius: 8px;
  font-size: clamp(0.8rem, 2vw, 1rem);
}

/* Modal Footer */
.modal-footer {
  padding: 1.5rem;
  border-top: 1px solid #e9ecef;
  background: #f8f9fa;
  border-radius: 0 0 20px 20px;
}

.whatsapp-btn {
  width: 100%;
  background: linear-gradient(135deg, #25d366, #128c7e);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 15px;
  font-weight: bold;
  font-size: clamp(1rem, 2.5vw, 1.1rem);
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.whatsapp-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(37, 211, 102, 0.4);
}

.whatsapp-btn:disabled {
  background: #95a5a6;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.whatsapp-btn span {
  font-size: 1.2rem;
}

/* Responsive Design */
@media (max-width: 768px) {
  .container {
    padding-bottom: 140px;
  }
  
  .header {
    padding: 1.5rem 1rem;
  }
  
  .menu {
    padding: 0.75rem;
  }
  
  .tab-content {
    padding: 1.5rem 0.75rem;
  }
  
  .items-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  
  .card-content {
    padding: 1rem;
  }
  
  .price-section {
    flex-direction: column;
    align-items: stretch;
    gap: 0.75rem;
  }
  
  .quantity-control {
    justify-content: center;
  }
  
  .cart-summary {
    padding: 0.75rem;
    flex-direction: column;
    gap: 0.75rem;
    text-align: center;
  }
  
  .modal-overlay {
    padding: 0.5rem;
  }
  
  .modal {
    max-height: 95vh;
  }
  
  .modal-header,
  .modal-content,
  .modal-footer {
    padding: 1rem;
  }
  
  .form-row {
    grid-template-columns: 1fr;
    gap: 0.75rem;
  }
  
  .payment-options {
    gap: 0.5rem;
  }
  
  .payment-option {
    padding: 0.75rem;
  }
}

@media (max-width: 480px) {
  .container {
    padding-bottom: 160px;
  }
  
  .header {
    padding: 1rem 0.75rem;
  }
  
  .menu {
    padding: 0.5rem;
  }
  
  .menu-tabs {
    gap: 0.25rem;
  }
  
  .tab-button {
    padding: 0.5rem 1rem;
    font-size: 0.8rem;
  }
  
  .tab-content {
    padding: 1rem 0.5rem;
  }
  
  .card-content {
    padding: 0.75rem;
  }
  
  .cart-summary {
    padding: 0.5rem;
  }
  
  .checkout-btn {
    padding: 0.75rem 1.5rem;
    font-size: 0.9rem;
  }
  
  .modal-header,
  .modal-content,
  .modal-footer {
    padding: 0.75rem;
  }
  
  .whatsapp-btn {
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
  }
}

/* Animações */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card {
  animation: fadeIn 0.5s ease-out;
}

.modal {
  animation: fadeIn 0.3s ease-out;
}

/* Scrollbar personalizada */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, #ff6b6b, #ee5a24);
  border-radius: 10px;
}

::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(135deg, #ee5a24, #ff6b6b);
}

/* Estados de loading e erro */
.loading {
  text-align: center;
  padding: 2rem;
  color: white;
  font-size: 1.2rem;
}

.error {
  text-align: center;
  padding: 2rem;
  color: #e74c3c;
  background: white;
  border-radius: 10px;
  margin: 1rem;
}

/* Acessibilidade */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Focus visible para navegação por teclado */
.tab-button:focus-visible,
.qty-btn:focus-visible,
.checkout-btn:focus-visible,
.whatsapp-btn:focus-visible {
  outline: 3px solid #4dabf7;
  outline-offset: 2px;
}

/* Print styles */
@media print {
  .cart-summary,
  .modal-overlay {
    display: none !important;
  }
}
</style>
