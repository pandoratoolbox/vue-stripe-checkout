<template>
  <div>
    <form id="payment-form">
      <slot name="card-element">
        <div id="card-element"></div>
      </slot>
      <slot name="card-errors">
        <div id="card-errors" role="alert"></div>
      </slot>
      <button ref="submitButtonRef" type="submit" class="hide"></button>
    </form>
  </div>
</template>

<script>
import { loadStripeCheckout } from './load-checkout';
export default {
  props: {
    pk: {
      type: String,
      required: true,
    },
    clientSecret: {
      type: String,
    },
    stripeAccount: {
      type: String,
    },
    apiVersion: {
      type: String,
    },
    locale: {
      type: String,
      default: 'auto',
    },
    styleObject: {
      type: Object,
    }
  },
  data () {
    return {
      loading: false,
      stripe: null,
      elements: null,
      card: null,
    };
  },
  computed: {
    style () {
      return {
        base: {
          color: '#32325d',
          fontFamily: '"Helvetica Neue", Helvetica, sans-serif',
          fontSmoothing: 'antialiased',
          fontSize: '16px',
          '::placeholder': {
            color: '#aab7c4'
          }
        },
        invalid: {
          color: '#fa755a',
          iconColor: '#fa755a'
        }
      };
    },
    form () {
      return document.getElementById('payment-form');
    },
  },
  methods: {
    submit () {
      console.log("submitted")
      this.$refs.submitButtonRef.click();
    },
  },
  mounted () {
    loadStripeCheckout(this.pk, 'v3', () => {
      const options = {
        stripeAccount: this.stripeAccount,
        apiVersion: this.apiVersion,
        locale: this.locale,
      };
      console.log("test")
      console.error("test 2")
      this.stripe = window.Stripe(this.pk, options);
      this.elements = this.stripe.elements();
      this.card = this.elements.create('card', { style: this.styleObject || this.style });
      this.card.mount('#card-element');

      this.card.addEventListener('change', ({ error }) => {
        const displayError = document.getElementById('card-errors');
        if (error) {
          displayError.textContent = error.message;
          return;
        }
        displayError.textContent = '';
      });
      
      this.form.addEventListener('submit', async (event) => {
        console.log("submitted")
        try {

          this.$emit('loading', true);
          event.preventDefault();
          const data = {
            ...this.card
          };
          if (this.amount) data.amount = this.amount;
          if (!this.client_secret) console.error("No client secret."); return;
          const { paymentIntent, error } = await this.stripe.confirmCardPayment(this.client_secret, { payment_method: {
            card: data
          }});
          if (error) {
            const errorElement = document.getElementById('card-errors');
            errorElement.textContent = error.message;
            console.error(error);
            this.$emit('error 1', error);
            return;
          }
          this.$emit('paymentIntent', paymentIntent);
        } catch (error) {
          console.error(error);
          this.$emit('error', error);
        } finally {
          this.$emit('loading', false);
        }
      });
    });
  }
}
</script>

<style scoped>
.StripeElement {
  box-sizing: border-box;

  height: 40px;

  padding: 10px 12px;

  border: 1px solid transparent;
  border-radius: 4px;
  background-color: white;

  box-shadow: 0 1px 3px 0 #e6ebf1;
  -webkit-transition: box-shadow 150ms ease;
  transition: box-shadow 150ms ease;
}

.StripeElement--focus {
  box-shadow: 0 1px 3px 0 #cfd7df;
}

.StripeElement--invalid {
  border-color: #fa755a;
}

.StripeElement--webkit-autofill {
  background-color: #fefde5 !important;
}

.hide {
  visibility: hidden;
}
</style>
