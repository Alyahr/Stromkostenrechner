<script setup>
import { computed, ref } from 'vue'

// Form inputs
const deviceA = ref('')
const deviceB = ref('')
const unit = ref('watt')
const hours = ref('')
const minutes = ref('')
const daysPerWeek = ref(7)
const pricePerKwh = ref(0.33)
const showResults = ref(false)

// Computed values
const unitPlaceholder = computed(() =>
  unit.value === 'watt' ? "z.B. '180' Watt" : "z.B. '1,27' kWh",
)

const daysOptions = [
  { title: '7 Tage', value: 7 },
  { title: '6 Tage', value: 6 },
  { title: '5 Tage', value: 5 },
  { title: '4 Tage', value: 4 },
  { title: '3 Tage', value: 3 },
  { title: '2 Tage', value: 2 },
  { title: '1 Tag', value: 1 },
]

const isFormValid = computed(() => {
  if (!deviceA.value) return false
  if (unit.value === 'watt' && !hours.value && !minutes.value) return false
  if (!pricePerKwh.value) return false
  return true
})

// Calculation results
const yearlyKwhA = ref(0)
const yearlyKwhB = ref(0)
const yearlyCostA = ref(0)
const yearlyCostB = ref(0)

// Methods
function calculateYearlyKwh(device) {
  if (!device) return 0

  const dailyHours = Number(hours.value) + Number(minutes.value) / 60

  if (unit.value === 'watt') {
    return (device * dailyHours * daysPerWeek.value * 52) / 1000
  } else {
    return device * daysPerWeek.value * 52
  }
}

function calculateCosts() {
  yearlyKwhA.value = calculateYearlyKwh(Number(deviceA.value))
  yearlyKwhB.value = calculateYearlyKwh(Number(deviceB.value))

  yearlyCostA.value = yearlyKwhA.value * Number(pricePerKwh.value)
  yearlyCostB.value = yearlyKwhB.value * Number(pricePerKwh.value)

  showResults.value = true
}
</script>

<template>
  <v-container>
    <v-card elevation="3">
      <v-card-title class="text-h4 text-center py-6 bg-primary text-white">
        Stromkostenrechner
      </v-card-title>

      <v-card-text class="pa-6">
        <v-form @submit.prevent="calculateCosts">
          <!-- Geräte Eingabe -->
          <v-card class="mb-6" variant="outlined">
            <v-card-title class="text-h6 px-4 pt-4">
              <v-icon icon="mdi-lightning-bolt" class="me-2"></v-icon>
              Geräte
            </v-card-title>
            <v-card-text>
              <v-text-field
                v-model="deviceA"
                label="Gerät A"
                type="number"
                step="0.01"
                :placeholder="unitPlaceholder"
                :rules="[(v) => !!v || 'Gerät A ist erforderlich']"
                variant="outlined"
                density="comfortable"
              ></v-text-field>

              <v-text-field
                v-model="deviceB"
                label="Gerät B (optional)"
                type="number"
                step="0.01"
                :placeholder="unitPlaceholder"
                variant="outlined"
                density="comfortable"
              ></v-text-field>

              <v-radio-group v-model="unit" inline class="mt-2">
                <v-radio label="Watt" value="watt"></v-radio>
                <v-radio label="kWh" value="kwh"></v-radio>
              </v-radio-group>
            </v-card-text>
          </v-card>

          <!-- Zeiteinstellung -->
          <v-card class="mb-6" variant="outlined">
            <v-card-title class="text-h6 px-4 pt-4">
              <v-icon icon="mdi-clock-outline" class="me-2"></v-icon>
              Nutzungsdauer
            </v-card-title>
            <v-card-text>
              <v-row>
                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="hours"
                    label="Stunden pro Tag"
                    type="number"
                    min="0"
                    max="24"
                    variant="outlined"
                    density="comfortable"
                    :rules="[
                      (v) =>
                        unit === 'watt' && hours === '' && minutes === ''
                          ? 'Nutzungsdauer erforderlich'
                          : true,
                    ]"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="minutes"
                    label="Minuten pro Tag"
                    type="number"
                    min="0"
                    max="59"
                    variant="outlined"
                    density="comfortable"
                  ></v-text-field>
                </v-col>
              </v-row>

              <v-select
                v-model="daysPerWeek"
                :items="daysOptions"
                label="Tage pro Woche"
                variant="outlined"
                density="comfortable"
              ></v-select>
            </v-card-text>
          </v-card>

          <!-- Strompreis -->
          <v-card class="mb-6" variant="outlined">
            <v-card-title class="text-h6 px-4 pt-4">
              <v-icon icon="mdi-currency-eur" class="me-2"></v-icon>
              Strompreis
            </v-card-title>
            <v-card-text>
              <v-text-field
                v-model="pricePerKwh"
                label="Strompreis pro kWh (€)"
                type="number"
                step="0.01"
                variant="outlined"
                density="comfortable"
                :rules="[(v) => !!v || 'Strompreis ist erforderlich']"
              ></v-text-field>
            </v-card-text>
          </v-card>

          <v-btn
            block
            color="primary"
            size="large"
            type="submit"
            :disabled="!isFormValid"
            elevation="2"
            class="text-h6"
          >
            Berechnen
          </v-btn>
        </v-form>

        <!-- Ergebnisse -->
        <v-expand-transition>
          <v-card v-if="showResults" class="mt-6" variant="outlined">
            <v-card-title class="text-h6 px-4 pt-4">
              <v-icon icon="mdi-calculator" class="me-2"></v-icon>
              Ergebnisse
            </v-card-title>
            <v-card-text>
              <v-list>
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon icon="mdi-alpha-a-circle" color="primary"></v-icon>
                  </template>
                  <v-list-item-title class="text-h6">
                    Gerät A: {{ yearlyKwhA.toFixed(2) }} kWh
                  </v-list-item-title>
                  <v-list-item-subtitle class="text-h6 text-primary">
                    {{ yearlyCostA.toFixed(2) }} €
                  </v-list-item-subtitle>
                </v-list-item>

                <v-list-item v-if="deviceB">
                  <template v-slot:prepend>
                    <v-icon icon="mdi-alpha-b-circle" color="secondary"></v-icon>
                  </template>
                  <v-list-item-title class="text-h6">
                    Gerät B: {{ yearlyKwhB.toFixed(2) }} kWh
                  </v-list-item-title>
                  <v-list-item-subtitle class="text-h6 text-secondary">
                    {{ yearlyCostB.toFixed(2) }} €
                  </v-list-item-subtitle>
                </v-list-item>
              </v-list>

              <v-card v-if="deviceB" class="mt-4" color="success-lighten-5">
                <v-card-title class="text-h6">
                  <v-icon icon="mdi-chart-line" class="me-2"></v-icon>
                  Vergleich
                </v-card-title>
                <v-card-text>
                  <div class="text-h6 mb-2">
                    Jährliche Ersparnis:
                    <span class="text-success">{{ (yearlyCostA - yearlyCostB).toFixed(2) }} €</span>
                  </div>
                  <div class="text-h6">
                    Ersparnis nach 10 Jahren:
                    <span class="text-success">
                      {{ ((yearlyCostA - yearlyCostB) * 10).toFixed(2) }} €
                    </span>
                  </div>
                </v-card-text>
              </v-card>
            </v-card-text>
          </v-card>
        </v-expand-transition>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<style scoped>
.v-card-title {
  font-weight: 600;
}
</style>
