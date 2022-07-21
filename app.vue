<template>
	<v-app>
		<v-app-bar app color="primary" dark>
			<v-toolbar-title>Leasing Kalkulator</v-toolbar-title>
		</v-app-bar>

		<v-main>
			<v-container fluid>
				<v-row>
					<v-col cols="12">
						<v-row align="center" justify="center">
							<v-col cols="12" md="6"></v-col>
						</v-row>
						<v-row align="center" justify="center">
							<v-col cols="12" md="6">
								<v-card raised>
									<v-col>
										<v-form ref="form" v-model="data.valid" lazy-validation>
											<v-text-field
												v-model="data.initialAmount"
												label="wpłata własna brutto"
												variant="solo"
											></v-text-field>
											<v-text-field
												v-model="data.amount"
												label="Cena samochodu netto"
												:rules="[(v) => !!v || 'Cena jest wymagana']"
												required
												variant="solo"
											></v-text-field>
											<v-text-field
												v-model="data.leasingRate"
												label="Rata Leasingowa netto"
												:rules="[(v) => !!v || 'Rata jest wymagana']"
												variant="solo"
											></v-text-field>

											<div class="mt-4 d-flex">
												<v-btn color="primary" class="mr-4" @click="calculate"
													>Oblicz</v-btn
												>
												<v-btn class="mr-4" @click="reset">Wyczyść</v-btn>
											</div>
										</v-form>
									</v-col>
									<v-col v-if="data.showResults">
										<v-row>
											<v-col><h2>Wyniki</h2></v-col>
											<v-divider></v-divider>
										</v-row>
										<v-row>
											<v-col>Cena netto:</v-col>
											<v-col class="font-weight-bold">{{ netto }} pln</v-col>
											<v-divider></v-divider>
										</v-row>
										<v-row>
											<v-col>Cena brutto:</v-col>
											<v-col class="font-weight-bold">{{ brutto }} pln</v-col>
											<v-divider></v-divider>
										</v-row>
										<v-row>
											<v-col>VAT:</v-col>
											<v-col class="font-weight-bold">{{ vat }} pln</v-col>
											<v-divider></v-divider>
										</v-row>
										<v-row>
											<v-col>Rata netto:</v-col>
											<v-col class="font-weight-bold">{{ leasingRateNetto }} pln</v-col>
											<v-divider></v-divider>
										</v-row>
										<v-row>
											<v-col>Rata brutto:</v-col>
											<v-col class="font-weight-bold">{{ leasingRateBrutto }} pln</v-col>
											<v-divider></v-divider>
										</v-row>
										<v-row>
											<v-col> Faktyczny koszt raty:</v-col>
											<v-col class="font-weight-bold">{{ actualRateExpense }} pln</v-col>
											<v-divider></v-divider>
										</v-row>
										<v-row>
											<v-col> Faktyczny koszt wpłaty własnej:</v-col>
											<v-col class="font-weight-bold">{{ actualInitialExpense }} pln</v-col>
											<v-divider></v-divider>
										</v-row>
									</v-col>
								</v-card>
							</v-col>
						</v-row>
					</v-col>
				</v-row>
			</v-container>
		</v-main>
	</v-app>
</template>
<script setup lang="ts">
const data = reactive({
	valid: false,
	amount: '',
	amountType: 'netto',
	LIMIT: 150000,
	showResults: false,
	bottomNav: 'recent',
	leasingRate: '',
	TAX_RATE: 0.19,
	rateType: 'netto',
	insurance: '',
	initialAmount: '',
});
const form = ref(null);

const actualInitialExpense = computed(() =>
	data.initialAmount ? parseInt(data.initialAmount) * 0.66 : 0
);

const brutto = computed(() => {
	return data.amount ? parseInt((parseInt(data.amount) * 1.23).toFixed(0)) : 0;
});
const netto = computed(() => {
	return data.amount ? parseInt(data.amount) : 0;
});
const vat = computed(() => {
	const result = brutto.value - netto.value;
	return result ? result : 0;
});

const calulcatedHalfVAT = computed(() => {
	const halfVAT = vat.value / 2;
	return netto.value + halfVAT;
});

const overLimit = computed(() => {
	return calulcatedHalfVAT.value - data.LIMIT;
});

const checkIfIsOverLimit = computed(() => {
	return calulcatedHalfVAT.value > data.LIMIT;
});
const halfRateVAT = computed(() => {
	return (leasingRateBrutto.value - leasingRateNetto.value) / 2;
});
const leasingRateNetto = computed(() => {
	return data.leasingRate ? parseInt(data.leasingRate) : 0;
});
const leasingRateBrutto = computed(() => {
	return data.leasingRate ? parseInt(data.leasingRate) * 1.23 : 0;
});

const expanse = computed(() => leasingRateNetto.value + halfRateVAT.value);

const actualRateExpense = computed(() => {
	const result = expanse.value * data.TAX_RATE;
	return expanse.value && result
		? parseInt(expanse.value.toFixed(0)) - parseInt(result.toFixed(0))
		: 0;
});

async function calculate() {
	await form.value.validate();
	if (data.valid) {
		data.showResults = true;
	} else {
		data.showResults = false;
	}
}
function reset() {
	form.value.reset();
	data.showResults = false;
}
function resetValidation() {
	data.form.resetValidation();
}
</script>
