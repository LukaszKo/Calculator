<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-toolbar-title>Calculator</v-toolbar-title>
    </v-app-bar>

    <v-content>
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
                    <v-form ref="form" v-model="valid" lazy-validation>
                      <v-row>
                        <v-col>
                          <v-text-field
                            v-model="amount"
                            label="Cena samochodu"
                            :rules="[v => !!v || 'Cena jest wymagana']"
                            required
                          ></v-text-field>
                        </v-col>
                        <v-col>
                          <v-radio-group
                            v-model="amountType"
                            row
                            :rules="[v => !!v || 'Zaznacz typ ceny']"
                            required
                          >
                            <v-radio label="netto" value="netto"></v-radio>
                            <v-radio label="brutto" value="brutto"></v-radio>
                          </v-radio-group>
                        </v-col>
                      </v-row>
                      <v-row>
                        <v-col>
                          <v-text-field
                            v-model="leasingRate"
                            label="Rata Leasingowa"
                            :rules="[v => !!v || 'Rata jest wymagana']"
                          ></v-text-field>
                        </v-col>
                        <v-col>
                          <v-radio-group
                            v-model="rateType"
                            row
                            :rules="[v => !!v || 'Zaznacz typ raty']"
                            required
                          >
                            <v-radio label="netto" value="netto"></v-radio>
                            <v-radio label="brutto" value="brutto"></v-radio>
                          </v-radio-group>
                        </v-col>
                      </v-row>
                      <v-text-field
                        v-model="insurance"
                        type="number"
                        label="Ubezpieczenie za rok (% od ceny netto)"
                      ></v-text-field>
                      <div class="mt-4">
                        <v-btn
                          :disabled="!valid"
                          color="success"
                          class="mr-4"
                          @click="calculate"
                        >Oblicz</v-btn>
                        <v-btn color="error" class="mr-4" @click="reset">Wyczyść</v-btn>
                      </div>
                    </v-form>
                  </v-col>
                  <div v-if="showResults">
                    <v-card-title primary-title>Wyniki</v-card-title>
                    <v-card-text>
                      Cena NETTO:
                      <span class="blue--text">{{netto | toFixed | currency}}</span>
                    </v-card-text>
                    <v-card-text>
                      Cena BRUTTO:
                      <span class="blue--text">{{brutto | toFixed | currency}}</span>
                    </v-card-text>
                    <v-card-text>
                      VAT:
                      <span class="blue--text">{{vat | toFixed | currency}}</span>
                    </v-card-text>
                    <v-card-text>
                      Rata NETTO:
                      <span class="blue--text">{{leasingRateNetto | toFixed | currency}}</span>
                    </v-card-text>
                    <v-card-text>
                      Rata BRUTTO:
                      <span
                        class="blue--text"
                      >{{leasingRateBrutto | toFixed | currency}}</span>
                    </v-card-text>
                    <v-card-text>
                      Faktyczny koszt raty:
                      <span
                        class="blue--text"
                      >{{actualRateExpense | toFixed | currency}}</span>
                    </v-card-text>
                    <v-card-text>
                      Ubezpieczenie za rok:
                      <span
                        class="blue--text"
                      >{{insuranceValue | toFixed | currency}}</span>
                    </v-card-text>
                    <v-card-text v-if="checkIfIsOverLimit">
                      Kwota ponad limit:
                      <span class="blue--text">{{overLimit | toFixed | currency}}</span>
                    </v-card-text>
                  </div>
                </v-card>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    valid: false,
    amount: "",
    amountType: "",
    LIMIT: 150000,
    showResults: false,
    bottomNav: "recent",
    leasingRate: "",
    TAX_RATE: 0.19,
    rateType: "",
    insurance: ""
  }),
  filters: {
    currency(val) {
      if (!val) return "";
      return `${val} pln`;
    },
    toFixed(val) {
      if (!val) return "";
      return parseInt(val).toFixed(2);
    }
  },
  computed: {
    brutto() {
      return this.amountType === "netto"
        ? (parseInt(this.amount) * 1.23).toFixed(0)
        : this.amount;
    },
    netto() {
      return this.amountType === "brutto"
        ? (parseInt(this.amount) / 1.23).toFixed(0)
        : this.amount;
    },
    vat() {
      return this.brutto - this.netto;
    },
    calulcatedHalfVAT() {
      const halfVAT = parseInt(this.vat) / 2;
      return parseInt(this.netto) + halfVAT;
    },
    overLimit() {
      return this.calulcatedHalfVAT - this.LIMIT;
    },
    checkIfIsOverLimit() {
      return this.calulcatedHalfVAT > this.LIMIT;
    },
    halfRateVAT() {
      return parseInt((this.leasingRateBrutto - this.leasingRateNetto) / 2);
    },
    leasingRateNetto() {
      return this.rateType === "brutto"
        ? (parseInt(this.leasingRate) / 1.23).toFixed(2)
        : parseInt(this.leasingRate);
    },
    leasingRateBrutto() {
      return this.rateType === "netto"
        ? parseInt(this.leasingRate) * 1.23
        : parseInt(this.leasingRate);
    },
    actualRateExpense() {
      const expense = this.leasingRateNetto + this.halfRateVAT;
      const result = expense * this.TAX_RATE;
      return expense - parseInt(result);
    },
    insuranceValue() {
      return (this.netto * this.insurance) / 100;
    }
  },
  methods: {
    async calculate() {
      await this.$refs.form.validate();
      if (this.valid) {
        this.showResults = true;
      } else {
        this.showResults = false;
      }
    },
    reset() {
      this.$refs.form.reset();
      this.showResults = false;
    },
    resetValidation() {
      this.$refs.form.resetValidation();
    }
  }
};
</script>
