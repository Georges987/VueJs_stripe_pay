<template>
    <div>
        <form @submit.prevent="handleSubmit">
            <div id="card-element" class="card-box"></div>
            <button type="submit" :disabled="loading">
                {{ loading ? 'Paiement...' : 'Payer' }}
            </button>
        </form>
        <p v-if="message">{{ message }}</p>
    </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { loadStripe } from '@stripe/stripe-js';

const stripePromise = loadStripe(import.meta.env.VITE_STRIPE_PUBLISHABLE_KEY);
const stripe = ref(null);
const elements = ref(null);
const card = ref(null);
const loading = ref(false);
const message = ref('');

onMounted(async () => {
    stripe.value = await stripePromise;
    elements.value = stripe.value.elements();
    card.value = elements.value.create('card');
    card.value.mount('#card-element');
});

const handleSubmit = async () => {
    loading.value = true;
    message.value = '';

    // Appelle ton backend Laravel pour créer un PaymentIntent
    const res = await fetch('http://localhost:8000/api/pay', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
            // Authorization: 'Bearer TOKEN' si besoin
        },
        body: JSON.stringify({ amount: 1000 }) // Montant en centimes = 10€
    });

    const data = await res.json();

    const result = await stripe.value.confirmCardPayment(data.clientSecret, {
        payment_method: {
            card: card.value
        }
    });

    if (result.error) {
        message.value = 'Erreur : ' + result.error.message;
    } else if (result.paymentIntent.status === 'succeeded') {
        message.value = '✅ Paiement réussi !';
    }

    loading.value = false;
};
</script>

<style scoped>
.card-box {
    border: 1px solid #ccc;
    padding: 12px;
    border-radius: 6px;
    margin-bottom: 16px;
}
</style>
