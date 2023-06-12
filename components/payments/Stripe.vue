<template>
	<div>
		<loading :active.sync="isLoading" :color="color" :background-color="backgroundColor">
		</loading>
		<div class="columns is-variable is-3 is-desktop">
			<div class="column is-9">
				<div class="column is-3">
					<vue-select v-model="selected_plan_option" :options="planOptions" :placeholder="selected_plan_option"
						@input="selectPlanOption" :clearable="false" />
				</div>
				<div class="columns is-multiline is-variable is-7 p-3">
					<div class="column is-4" v-for="plan in plans" :key="plan.id"
						v-if="(selected_plan_option == 'Yearly' && plan.nickname.indexOf('Y') != -1) || selected_plan_option == 'Monthly' && plan.nickname.indexOf('M') != -1">
						<div class="card has-background-white has-text-black">
							<div class="card-content  has-text-centered">
								<div class="is-size-6 has-text-weight-bold has-text-black has-text-weight-regular mb-3 mt-5">
									{{ plan.title }}
								</div>
								<div class="is-size-3 has-text-weight-bold has-text-primary ">
									{{
										currency_symbol +
										((plan.amount * currency_rate) / 100).toFixed(2)
									}}
									<p class="is-size-6 has-text-weight-bold has-text-primary ">
										({{ selected_plan_option }})
									</p>
								</div>
								<div class="is-size-6 has-text-black is-uppercase has-text-weight-bold mt-5">
									{{ plan.nickname }}
								</div>
								<div class="mt-3 mb-5" style="line-height: 2rem;">
									{{ plan.description }}
								</div>
								<NuxtLink v-if="has_payment_method == false" :to="'/subscription/stripe?name=' + plan.id"
									class="button is-size-7 is-uppercase has-text-white has-background-primary has-text-weight-medium is-light mt-4">
									Subscribe by card</NuxtLink>
								<NuxtLink v-if="false && has_payment_method == false" :to="'/subscription/paypal?name=' + plan.id"
									class="button is-size-7 is-uppercase has-text-white has-background-primary has-text-weight-medium is-light mt-4">
									Subscribe by PayPal</NuxtLink>

								<div v-if="has_payment_method && plan.subscribed === false">
									<button @click="open(plan.id)"
										class="button is-size-7 is-uppercase has-text-white has-background-primary has-text-weight-medium is-light mt-4">
										Subscribe
									</button>
								</div>
								<div v-if="plan.subscribed">
									<button @click="open(null)"
										class="button is-size-7 is-uppercase is-danger has-text-weight-medium is-light mt-4"
										:class="{ 'is-success': plan.subscribed }">
										Unsubscribe
									</button>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="column is-3">
				<div class="columns is-gapless is-multiline">
					<div class="card plan_info_block has-background-primary has-text-black">
						<img src="~assets/images/plan_info_img.svg" alt="" />
						<div class="card-content">
							<div class="has-text-black has-text-weight-medium is-flex plans_info mb-5">
								<i class="fas fa-check mr-2 mt-1"></i>
								<p class="is-size-7">
									Family Tree 365 is a secure online website which you can use
									to create your own family tree(s) with.
								</p>
							</div>
							<div class="has-text-black has-text-weight-medium is-flex plans_info mb-5">
								<i class="fas fa-check mr-2 mt-1"></i>
								<p class="is-size-7">
									It has a tree viewer and DNA support more features are planned
									such as the inclusion of archive databases & collections
								</p>
							</div>
							<div class="has-text-black has-text-weight-medium is-flex plans_info">
								<i class="fas fa-check mr-2 mt-1"></i>
								<p class="is-size-7">
									It is aimed to be affordable with a 7 day no obligation trial
									with different pricing levels depending on how many trees you
									require.
								</p>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
		<div class="modal" :class="{ 'is-active': isActive }">
			<div class="modal-background"></div>
			<div class="modal-card">
				<header class="modal-card-head">
					<p class="modal-card-title">Confirmation</p>
					<button class="delete" aria-label="close" @click="close()"></button>
				</header>
				<section class="modal-card-body">Are you sure?</section>
				<footer class="modal-card-foot">
					<button class="button is-success" @click="subscribe()" v-if="selectedPlanId != null">
						Subscribe
					</button>
					<button class="button is-success" @click="unsubscribe()" v-else>
						Unsubscribe
					</button>
					<button class="button" @click="close()">No</button>
				</footer>
			</div>
		</div>
	</div>
</template>

<script>
import 'vue-loading-overlay/dist/vue-loading.css'
import VueSwitch from '@enso-ui/switch/bulma'
import vSelect from 'vue-select'
import Loading from 'vue-loading-overlay'
import { mapGetters, mapActions } from 'vuex'
const plandata = [
	{
		title: 'Premium',
		id: 0,
		amount: 10,
		nickname: 'UTY',
		paypal_id: 0,
		description: `
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
		`
	},
	{
		title: 'Premium',
		id: 1,
		amount: 10,
		nickname: 'OTM',
		paypal_id: 0,
		description: `
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
		`
	},
	{
		title: 'Basic',
		id: 3,
		amount: 10,
		nickname: 'TTY',
		paypal_id: 0,
		description: `
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
		`
	},
	{
		title: 'Basic',
		id: 4,
		amount: 10,
		nickname: 'TTM',
		paypal_id: 0,
		description: `
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
		`
	},
	{
		title: 'Starter',
		id: 5,
		amount: 10,
		nickname: 'OTY',
		paypal_id: 0,
		description: `
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
		`
	},
	{
		title: 'Starter',
		id: 2,
		amount: 10,
		nickname: 'UTM',
		paypal_id: 0,
		description: `
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
			Pricing tier feature will be placed here. \n 
		`
	},
]
export default {
	props: ['currency', 'currency_symbol', 'currency_rate'],
	components: {
		vSelect,
		Loading,
		VueSwitch,
	},
	data() {
		return {
			error: false,
			message: '',
			isLoading: false,
			fullPage: true,
			color: '#4fcf8d',
			backgroundColor: '#ffffff',
			response: null,
			has_payment_method: false,
			plans: [],
			selectedPlanId: null,
			isActive: false,
			planOptions: ['Yearly', 'Monthly'],
			selected_plan_option: 'Yearly'
		}
	},
	computed: {
		...mapGetters(['loggedInUser'])
	},
	methods: {
		async getCurrentSubscription() {
			const response = await this.$axios.$get(
				'/api/stripe/current-subscription'
			)
			this.has_payment_method = response.has_payment_method
			if (response.subscribed) {
				// this.plans
				//     .find(plan => plan.id === response.plan_id)
				//     .subscribed = true;
				// this.plans
				//     .find(plan => plan.id !== response.plan_id)
				//     .subscribed = false;
				this.plans.find(plan => {
					if (plan.id == response.plan_id) {
						plan.subscribed = true
					} else {
						plan.subscribed = false
					}
				})
			} else {
				this.plans.map(plan => {
					plan.subscribed = false
				})
			}
		},
		selectPlanOption(option) {
			this.selected_plan_option = option
		},
		async loadPlans() {
			// const response = await this.$axios.$get('/api/stripe/plans')
			console.log("invoke loadplan")

			this.getCurrentSubscription()
			this.plans = plandata
			console.log("Plan Data", plandata)
			this.isLoading = false
		},
		handleSelectedFiles(event) {
			this.file = this.$refs.fileInput.files[0]
			this.fileName = this.file.name
		},
		submit() { },

		subscribe() {
			this.isLoading = false
			this.isActive = false
			this.$axios.$post('/api/stripe/subscribe', {
				plan_id: this.selectedPlanId
			}).then(() => {
				this.getCurrentSubscription()
				this.isLoading = false
			})
		},
		unsubscribe() {
			this.isLoading = false
			this.isActive = false
			this.$axios.post('/api/stripe/unsubscribe', {})
				.then(() => {
					this.getCurrentSubscription()
					this.isLoading = false
				})
		},
		open(planId) {
			this.isActive = true
			this.selectedPlanId = planId
		},
		close() {
			this.isActive = false
		}
	},
	created() {
		console.log("created")
		this.loadPlans()

	}
}
</script>