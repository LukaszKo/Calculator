<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center justify-center">Leasing Calculator</div>
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
                      <v-text-field
                        v-model="amount"
                        label="Kwota"
                        :rules="[v => !!v || 'Kwota jest wymagana']"
                        required
                      ></v-text-field>
                      <v-radio-group
                        v-model="type"
                        row
                        :rules="[v => !!v || 'Zaznacz typ kwoty']"
                        required
                      >
                        <v-radio label="netto" value="netto"></v-radio>
                        <v-radio label="brutto" value="brutto"></v-radio>
                      </v-radio-group>
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
                      NETTO:
                      <span class="blue--text">{{netto | currency}}</span>
                    </v-card-text>
                    <v-card-text>
                      BRUTTO:
                      <span class="blue--text">{{brutto | currency}}</span>
                    </v-card-text>
                    <v-card-text>
                      VAT:
                      <span class="blue--text">{{vat | currency}}</span>
                    </v-card-text>
                    <v-card-text v-if="checkIfIsOverLimit">
                      Kwota ponad limit:
                      <span class="blue--text">{{overLimit | currency}}</span>
                    </v-card-text>
                  </div>
                </v-card>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
    <!-- <v-bottom-navigation v-model="bottomNav">
      <v-btn value="recent">
        <span>Recent</span>
        <v-icon>mdi-history</v-icon>
      </v-btn>

      <v-btn value="favorites">
        <span>Favorites</span>
        <v-icon>mdi-heart</v-icon>
      </v-btn>

      <v-btn value="nearby">
        <span>Nearby</span>
        <v-icon>mdi-map-marker</v-icon>
      </v-btn>
    </v-bottom-navigation> -->
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    valid: false,
    amount: "",
    type: "",
    LIMIT: 150000,
    showResults: false,
    bottomNav: "recent"
  }),
  filters: {
    currency(val) {
      if (!val) return "";
      return `${val} pln`;
    }
  },
  computed: {
    brutto() {
      return this.type === "netto"
        ? (parseInt(this.amount) * 1.23).toFixed(0)
        : this.amount;
    },
    netto() {
      return this.type === "brutto"
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
