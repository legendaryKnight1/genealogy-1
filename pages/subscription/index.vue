<template>
  <div>
    <NuxtLink
      class="is-size-7 has-text-weight-medium has-text-link"
      to="/subscription/paypal"
      >PayPal</NuxtLink
    >
    <div class="currency-div">
			<span>Currency: </span>
			<div class="currency-inside-div">
				<vue-select :options="currency_options" :placeholder="selected_currency" @input="selectCurrency"
					:clearable="false" />
			</div>
		</div>
		<div class="column is-12">
			<h1 class="is-size-4 has-text-black">
				<span class="has-text-weight-medium">Subscription</span>
			</h1>
		</div>
		<div class="column is-12">
			<nav class="breadcrumb mt-1 mb-0" aria-label="breadcrumbs">
				<ul>
					<li>
						<NuxtLink class="is-size-7 has-text-weight-medium has-text-link" to="dashboard">Home</NuxtLink>
					</li>
					<li class="is-size-7 has-text-weight-medium is-active">
						<a href="" aria-current="page">Subscription</a>
					</li>
					<li class="is-size-7 has-text-weight-medium is-active">
						<a href="" aria-current="page">Stripe</a>
					</li>
				</ul>
			</nav>
		</div>
    <Stripe :currency_symbol="selected_currency_symbol" :currency="selected_currency" :currency_rate="selected_currency_rate"></Stripe>
  </div>
</template>
<router>
{
    name: 'subscription.index'
}
</router>

<script>
import 'vue-loading-overlay/dist/vue-loading.css'
import Stripe from '~/components/payments/Stripe.vue'
import { mapGetters, mapActions } from 'vuex'

export default {
  layout: 'default',
  head: {
    title: 'Subscription'
  },
  meta: {
    breadcrumb: 'subscription',
    title: 'Subscription'
  },
  components: {
    Stripe
  },
  data() {
    return {
      error: false,
      selected_currency: 'GBP',
			selected_currency_symbol: '£',
			selected_currency_rate: 1,
      currency_options: ['USD', 'GBP', 'EUR', 'AUD'],
    }
  },
  methods: {
    async selectCurrency(currency) {
			await fetch(
				'https://api.freecurrencyapi.com/v1/latest?apikey=9WkmXwTgkCNODiQbpgzXrgt1SZkSBsIA1B3xZyMe'
			)
				.then(response => response.json())
				.then(({ data }) => {
					console.log('new cur', data)
					switch (currency) {
						case 'GBP':
							this.selected_currency_symbol = '£'
							this.selected_currency_rate = 1
							break
						case 'USD':
							this.selected_currency_symbol = '$'
							this.selected_currency_rate = 1 / data.GBP
							break
						case 'EUR':
							this.selected_currency_symbol = '€'
							this.selected_currency_rate = data.EUR / data.GBP
							break
						case 'AUD':
							this.selected_currency_symbol = '$'
							this.selected_currency_rate = data.AUD / data.GBP
							break
						default:
							this.selected_currency_symbol = '£'
							this.selected_currency_rate = 1
							break
					}
				})
				.catch((err) => { console.log("error:",err) })
		},
  }
 
}
</script>
