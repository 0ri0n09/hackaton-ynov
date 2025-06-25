<template>
  <div class="min-h-screen transition-all duration-1000 relative overflow-hidden" :class="backgroundClass">
    <!-- Effets de fond animés -->
    <div class="absolute inset-0 pointer-events-none">
      <!-- Particules de pollution -->
      <div v-if="impactLevel >= 2" class="absolute inset-0">
        <div v-for="i in pollutionParticles" :key="i"
             class="absolute w-2 h-2 bg-gray-600 rounded-full opacity-30 animate-float"
             :style="{
               left: Math.random() * 100 + '%',
               top: Math.random() * 100 + '%',
               animationDelay: Math.random() * 3 + 's'
             }">
        </div>
      </div>

      <!-- Fumée pour impact élevé -->
      <div v-if="impactLevel >= 3" class="absolute bottom-0 left-0 right-0 h-32 bg-gradient-to-t from-gray-800/30 to-transparent animate-pulse"></div>

      <!-- Oiseaux pour faible impact -->
      <div v-if="impactLevel === 0" class="absolute top-20 left-10 animate-bounce">
        <div class="text-2xl">🐦</div>
      </div>
      <div v-if="impactLevel === 0" class="absolute top-32 right-20 animate-bounce" style="animation-delay: 1s">
        <div class="text-xl">🦋</div>
      </div>

      <!-- Fleurs qui fanent pour impact moyen -->
      <div v-if="impactLevel === 1" class="absolute bottom-20 left-20 opacity-60">
        <div class="text-2xl">🌻</div>
      </div>
      <div v-if="impactLevel === 1" class="absolute bottom-32 right-32 opacity-40">
        <div class="text-xl">🌸</div>
      </div>
    </div>

    <Header />
    <RandomFacts />

    <!-- Contenu principal -->
    <div class="container mx-auto px-4 py-8 relative z-10">
      <!-- Section Formulaire -->
      <section class="max-w-2xl mx-auto mb-12">
        <div class="bg-white/90 backdrop-blur-sm rounded-2xl shadow-xl p-6 sm:p-8 transition-all duration-500"
             :class="{ 'bg-gray-100/80': impactLevel >= 3 }">

          <!-- Formulaire -->
          <form @submit.prevent="calculateImpact" class="space-y-6">
            <!-- Input Kilomètres -->
            <div>
              <label for="kilometers" class="block text-lg font-semibold text-gray-700 mb-2">
                Combien de kilomètres veux-tu parcourir ?
              </label>
              <input
                  id="kilometers"
                  v-model.number="kilometers"
                  type="number"
                  min="0"
                  step="0.1"
                  placeholder="Ex: 25"
                  class="w-full px-4 py-3 text-xl border-2 border-green-300 rounded-lg focus:border-green-500 focus:outline-none transition-colors"
                  required
              >
            </div>

            <!-- Sélection du transport -->
            <div>
              <label class="block text-lg font-semibold text-gray-700 mb-4">
                Comment veux-tu voyager ?
              </label>
              <div class="grid grid-cols-2 md:grid-cols-3 gap-3 sm:gap-4">
                <button
                    v-for="transport in transports"
                    :key="transport.id"
                    type="button"
                    @click="selectedTransport = transport.id"
                    class="cursor-pointer p-3 sm:p-4 border-2 rounded-lg transition-all duration-200 hover:scale-105 focus:outline-none focus:ring-2 focus:ring-green-500"
                    :class="selectedTransport === transport.id
                    ? 'border-green-500 bg-green-50 shadow-lg'
                    : 'border-gray-300 hover:border-green-300'"
                >
                  <div class="text-2xl sm:text-3xl mb-2">{{ transport.emoji }}</div>
                  <div class="font-medium text-sm sm:text-base">{{ transport.name }}</div>
                  <div class="text-xs sm:text-sm text-gray-600">{{ transport.co2 }}g CO₂/km</div>
                </button>
              </div>
            </div>

            <!-- Bouton Calculer -->
            <button
                type="submit"
                :disabled="!kilometers || !selectedTransport"
                class="w-full py-3 sm:py-4 px-6 bg-gradient-to-r from-green-500 to-blue-500 text-white text-lg sm:text-xl font-bold rounded-lg hover:from-green-600 hover:to-blue-600 disabled:opacity-50 disabled:cursor-not-allowed transition-all duration-200 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-green-500 cursor-pointer"
            >
              🌍 Calculer mon empreinte
            </button>
          </form>
        </div>
      </section>

      <!-- Section Résultats -->
      <section v-if="result" class="max-w-2xl mx-auto">
        <div class="bg-white/90 backdrop-blur-sm rounded-2xl shadow-xl p-6 sm:p-8 transition-all duration-500 animate-fade-in"
             :class="resultCardClass">

          <!-- Résultat principal -->
          <div class="text-center mb-8">
            <div class="text-4xl sm:text-6xl mb-4 animate-bounce-slow">{{ impactEmoji }}</div>
            <h2 class="text-2xl sm:text-3xl font-bold mb-4" :class="resultTextClass">
              {{ result.co2 }} kg de CO₂
            </h2>
            <p class="text-base sm:text-lg text-gray-600 mb-4">
              Équivalent à {{ result.trees }} arbre{{ result.trees > 1 ? 's' : '' }} nécessaire{{ result.trees > 1 ? 's' : '' }} pour compenser
            </p>

            <!-- Barre de progression visuelle -->
            <div class="w-full bg-gray-200 rounded-full h-3 mb-4">
              <div class="h-3 rounded-full transition-all duration-1000"
                   :class="progressBarClass"
                   :style="{ width: Math.min(100, (result.co2 / 50) * 100) + '%' }">
              </div>
            </div>
          </div>

          <!-- Message de sensibilisation -->
          <div class="rounded-lg p-4 sm:p-6 mb-6 transition-all duration-500"
               :class="sensitizationCardClass">
            <p class="text-base sm:text-lg font-medium text-center mb-4" :class="messageTextClass">
              {{ sensitizationMessage }}
            </p>

            <!-- Suggestions -->
            <div v-if="suggestions.length > 0" class="space-y-2">
              <p class="font-semibold text-gray-700">💡 Suggestions :</p>
              <ul class="space-y-1">
                <li v-for="suggestion in suggestions" :key="suggestion" class="text-sm sm:text-base text-gray-600">
                  • {{ suggestion }}
                </li>
              </ul>
            </div>
          </div>

          <!-- Comparaisons -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-6">
            <div v-for="comparison in comparisons" :key="comparison.transport"
                 class="bg-green-50 rounded-lg p-3 sm:p-4 border border-green-200 hover:shadow-md transition-shadow">
              <div class="flex items-center justify-between">
                <span class="font-medium text-sm sm:text-base">{{ comparison.emoji }} {{ comparison.transport }}</span>
                <span class="text-green-600 font-bold text-sm sm:text-base">{{ comparison.co2 }} kg CO₂</span>
              </div>
            </div>
          </div>

          <!-- Actions -->
          <div class="flex flex-col sm:flex-row gap-4">
            <button
                @click="resetCalculator"
                class="flex-1 py-3 px-6 bg-gray-500 text-white font-medium rounded-lg hover:bg-gray-600 transition-colors focus:outline-none focus:ring-2 focus:ring-gray-500 cursor-pointer"
            >
              🔄 Nouveau calcul
            </button>
            <button
                @click="shareResult"
                class="flex-1 py-3 px-6 bg-blue-500 text-white font-medium rounded-lg hover:bg-blue-600 transition-colors focus:outline-none focus:ring-2 focus:ring-blue-500 cursor-pointer"
            >
              📤 Partager
            </button>
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import RandomFacts from "~/components/RandomFact.vue";

// État réactif
const kilometers = ref(0)
const selectedTransport = ref('')
const result = ref(null)

// Configuration des transports
const transports = [
  { id: 'walk', name: 'À pied', emoji: '🚶', co2: 0 },
  { id: 'bike', name: 'Vélo', emoji: '🚴', co2: 0 },
  { id: 'train', name: 'Train', emoji: '🚆', co2: 40 },
  { id: 'bus', name: 'Bus', emoji: '🚌', co2: 105 },
  { id: 'car', name: 'Voiture', emoji: '🚗', co2: 160 },
  { id: 'plane', name: 'Avion', emoji: '✈️', co2: 250 }
]

// Calcul du niveau d'impact
const impactLevel = computed(() => {
  if (!result.value) return 0
  const co2 = parseFloat(result.value.co2)
  if (co2 <= 1) return 0      // Très bas
  if (co2 <= 5) return 1      // Moyen
  if (co2 <= 20) return 2     // Élevé
  return 3                    // Très haut
})

// Classes CSS dynamiques pour le fond
const backgroundClass = computed(() => {
  const levels = [
    'bg-gradient-to-br from-green-100 via-blue-50 to-green-200', // Très bas
    'bg-gradient-to-br from-yellow-100 via-orange-50 to-yellow-200', // Moyen
    'bg-gradient-to-br from-gray-200 via-gray-300 to-gray-400', // Élevé
    'bg-gradient-to-br from-red-900 via-gray-800 to-black' // Très haut
  ]
  return levels[impactLevel.value] || levels[0]
})

// Classes pour les cartes de résultat
const resultCardClass = computed(() => {
  if (impactLevel.value >= 3) return 'bg-red-100/80 border-2 border-red-300'
  if (impactLevel.value >= 2) return 'bg-gray-100/80 border border-gray-300'
  return 'bg-green-50/80 border border-green-200'
})

const sensitizationCardClass = computed(() => {
  if (impactLevel.value >= 3) return 'bg-red-50 border border-red-200'
  if (impactLevel.value >= 2) return 'bg-orange-50 border border-orange-200'
  return 'bg-green-50 border border-green-200'
})

// Classes pour les textes
const resultTextClass = computed(() => {
  if (impactLevel.value >= 3) return 'text-red-700'
  if (impactLevel.value >= 2) return 'text-gray-700'
  return 'text-green-700'
})

const messageTextClass = computed(() => {
  if (impactLevel.value >= 3) return 'text-red-700'
  if (impactLevel.value >= 2) return 'text-orange-700'
  return 'text-green-700'
})

// Classe pour la barre de progression
const progressBarClass = computed(() => {
  if (impactLevel.value >= 3) return 'bg-red-500'
  if (impactLevel.value >= 2) return 'bg-orange-500'
  if (impactLevel.value >= 1) return 'bg-yellow-500'
  return 'bg-green-500'
})

// Emoji selon l'impact
const impactEmoji = computed(() => {
  const emojis = ['🌿', '🌾', '🌫️', '🔥']
  return emojis[impactLevel.value] || '🌿'
})

// Particules de pollution
const pollutionParticles = computed(() => {
  return Array.from({ length: impactLevel.value * 8 }, (_, i) => i)
})

// Messages de sensibilisation
const sensitizationMessage = computed(() => {
  if (!result.value) return ''

  const co2 = result.value.co2
  const messages = [
    `Bravo ! Tu as choisi un mode de transport écologique. ${co2} kg de CO₂, c'est presque rien ! 🌱`,
    `Pas mal ! Tu as laissé une trace de ${co2} kg de CO₂. Tu peux faire mieux ! 🌾`,
    `Attention ! Tu as laissé une trace de ${co2} kg de CO₂ aujourd'hui... mais tu peux en effacer une partie. 🌫️`,
    `Alerte rouge ! ${co2} kg de CO₂, c'est énorme ! Notre planète suffoque... 🔥`
  ]

  return messages[impactLevel.value] || messages[0]
})

// Suggestions
const suggestions = computed(() => {
  if (!result.value || impactLevel.value === 0) return []

  const allSuggestions = [
    'Et si tu faisais ce trajet à vélo demain ?',
    'Le train pourrait être une alternative plus verte',
    'Pense au covoiturage pour réduire ton impact',
    'Combine plusieurs trajets en un seul voyage',
    'Privilégie les transports en commun',
    'Organise tes déplacements pour optimiser tes trajets'
  ]

  return allSuggestions.slice(0, Math.min(3, impactLevel.value + 1))
})

// Comparaisons avec d'autres transports
const comparisons = computed(() => {
  if (!result.value) return []

  return transports
      .filter(t => t.id !== selectedTransport.value)
      .slice(0, 4)
      .map(transport => ({
        transport: transport.name,
        emoji: transport.emoji,
        co2: ((kilometers.value * transport.co2) / 1000).toFixed(2)
      }))
      .sort((a, b) => parseFloat(a.co2) - parseFloat(b.co2))
})

// Fonction de calcul
const calculateImpact = () => {
  const transport = transports.find(t => t.id === selectedTransport.value)
  if (!transport || !kilometers.value) return

  const co2Grams = kilometers.value * transport.co2
  const co2Kg = (co2Grams / 1000).toFixed(2)
  const trees = Math.max(1, Math.ceil(parseFloat(co2Kg) / 22)) // Un arbre absorbe ~22kg CO2/an

  result.value = {
    co2: co2Kg,
    trees: trees,
    transport: transport.name
  }

  // Scroll vers les résultats
  setTimeout(() => {
    const resultSection = document.querySelector('section:last-of-type')
    if (resultSection) {
      resultSection.scrollIntoView({ behavior: 'smooth' })
    }
  }, 100)
}

// Reset du calculateur
const resetCalculator = () => {
  kilometers.value = 0
  selectedTransport.value = ''
  result.value = null
}

// Partage des résultats
const shareResult = () => {
  if (!result.value) return

  const text = `J'ai calculé mon empreinte carbone : ${result.value.co2} kg de CO₂ pour ${kilometers.value} km en ${result.value.transport}. Et toi ? 🌍`

  if (navigator.share) {
    navigator.share({
      title: 'Mon Impact Carbone',
      text: text,
      url: window.location.href
    })
  } else {
    // Fallback pour copier dans le presse-papier
    navigator.clipboard.writeText(text).then(() => {
      alert('Résultat copié dans le presse-papier !')
    })
  }
}
</script>

<style scoped>
@keyframes float {
  0%, 100% {
    transform: translateY(0px) rotate(0deg);
    opacity: 0.3;
  }
  50% {
    transform: translateY(-20px) rotate(180deg);
    opacity: 0.6;
  }
}

@keyframes bounce-slow {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-float {
  animation: float 6s ease-in-out infinite;
}

.animate-bounce-slow {
  animation: bounce-slow 2s ease-in-out infinite;
}

.animate-fade-in {
  animation: fade-in 0.6s ease-out;
}

/* Transitions fluides */
.transition-all {
  transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Responsive container */
.container {
  max-width: 1200px;
}

/* Amélioration de l'accessibilité */
button:focus {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}

/* Effet de survol pour les cartes */
.hover\:shadow-md:hover {
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}
</style>