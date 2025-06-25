<script setup lang="ts">
import { ref, onMounted } from 'vue'

// État réactif
const fact = ref('')
const isLoading = ref(false)
const error = ref('')
const factCount = ref(0)

// Fonction pour générer un fait écologique
async function generateNewFact() {
  try {
    isLoading.value = true
    error.value = ''

    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer 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`,
      },
      body: JSON.stringify({
        model: 'gpt-4o-mini', // Modèle corrigé
        messages: [
          {
            role: 'system',
            content: `Tu es un expert en écologie et développement durable.
            Génère un fait écologique intéressant, surprenant ou utile en français.
            Le fait doit être :
            - Court (maximum 2 phrases)
            - Factuel et vérifiable
            - Inspirant ou motivant
            - Accessible au grand public
            - Lié à l'environnement, l'écologie, ou le développement durable

            Réponds uniquement avec le fait, sans guillemets ni introduction.`,
          },
          {
            role: 'user',
            content: 'Génère un nouveau fait écologique intéressant et surprenant.',
          },
        ],
        temperature: 0.8,
        max_tokens: 150,
      }),
    })

    if (!response.ok) {
      throw new Error(`Erreur HTTP: ${response.status}`)
    }

    const data = await response.json()

    if (data.choices && data.choices[0] && data.choices[0].message) {
      const generatedFact = data.choices[0].message.content.trim()
      fact.value = generatedFact.replace(/^["']|["']$/g, '') // Enlever les guillemets
      factCount.value += 1

      // Sauvegarder dans le localStorage
      localStorage.setItem('ecoFactCount', factCount.value.toString())
    } else {
      throw new Error('Réponse invalide de l\'API')
    }

  } catch (err) {
    console.error('Erreur lors de l\'appel à OpenAI :', err)
    error.value = err.message || 'Une erreur est survenue'

    // Faits de secours en cas d'erreur
    const fallbackFacts = [
      "Un arbre mature peut absorber jusqu'à 22 kg de CO₂ par an, soit l'équivalent des émissions d'une voiture sur 100 km.",
      "Les océans absorbent environ 25% du CO₂ que nous produisons, mais cela les rend plus acides et menace la vie marine.",
      "Recycler une tonne de papier permet d'économiser 17 arbres, 26 000 litres d'eau et 4 000 kWh d'électricité.",
      "Remplacer 10 km de trajet en voiture par du vélo évite l'émission de 2,6 kg de CO₂ dans l'atmosphère.",
      "Une douche de 4 minutes consomme 4 fois moins d'eau qu'un bain, soit une économie de 130 litres d'eau.",
      "Les abeilles pollinisent 1/3 de notre alimentation, mais leurs populations diminuent de 30% chaque année.",
      "Éteindre ses appareils en veille peut réduire sa facture électrique de 10% et éviter 86 kg de CO₂ par an.",
      "Les forêts tropicales produisent 20% de l'oxygène mondial mais disparaissent au rythme de 10 millions d'hectares par an."
    ]

    fact.value = fallbackFacts[Math.floor(Math.random() * fallbackFacts.length)]
    factCount.value += 1
    error.value = '' // Réinitialiser l'erreur puisqu'on a un fait de secours
  } finally {
    isLoading.value = false
  }
}

// Charger le compteur depuis le localStorage au montage
onMounted(() => {
  const savedCount = localStorage.getItem('ecoFactCount')
  if (savedCount) {
    factCount.value = parseInt(savedCount, 10)
  }

  // Générer un fait automatiquement au chargement
  generateNewFact()
})
</script>

<template>
  <div class="max-w-2xl mx-auto mb-8">
    <!-- Carte du fait écologique -->
    <div class="bg-gradient-to-br from-green-50 to-blue-50 rounded-2xl shadow-lg border border-green-200 p-6 relative overflow-hidden group hover:shadow-xl transition-shadow duration-300">
      <!-- Icône décorative -->
      <div class="absolute top-4 right-4 text-green-300 opacity-20">
        <div class="text-6xl">🌱</div>
      </div>

      <div class="flex items-center mb-4">
        <div class="bg-green-100 rounded-full p-2 mr-3 animate-pulse">
          <div class="text-2xl">💡</div>
        </div>
        <h3 class="text-xl font-bold text-green-800">Le saviez-vous ?</h3>
        <button
            @click="generateNewFact"
            :disabled="isLoading"
            class="ml-auto bg-green-500 hover:bg-green-600 disabled:opacity-50 disabled:cursor-not-allowed text-white rounded-full p-2 transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-green-500 hover:scale-105"
            title="Générer un nouveau fait"
        >
          <div class="text-sm" :class="{ 'animate-spin': isLoading }">🔄</div>
        </button>
      </div>

      <!-- Contenu du fait -->
      <div class="relative z-10">
        <!-- État de chargement -->
        <div v-if="isLoading" class="flex items-center justify-center py-8">
          <div class="flex items-center space-x-3">
            <div class="animate-spin rounded-full h-6 w-6 border-b-2 border-green-500"></div>
            <span class="text-green-700 font-medium">Génération d'un fait écologique...</span>
          </div>
        </div>

        <!-- État d'erreur -->
        <div v-else-if="error && !fact" class="bg-red-50 border border-red-200 rounded-lg p-4">
          <div class="flex items-center">
            <div class="text-red-500 mr-2">⚠️</div>
            <div>
              <p class="text-red-700 font-medium">Erreur lors du chargement</p>
              <p class="text-red-600 text-sm">{{ error }}</p>
            </div>
          </div>
          <button
              @click="generateNewFact"
              class="mt-3 bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg text-sm transition-colors focus:outline-none focus:ring-2 focus:ring-red-500"
          >
            Réessayer
          </button>
        </div>

        <!-- Fait écologique -->
        <div v-else-if="fact" class="space-y-4 animate-fade-in">
          <blockquote class="text-gray-700 leading-relaxed text-lg border-l-4 border-green-400 pl-4 py-2">
            <span class="text-green-600 text-2xl">"</span>{{ fact }}<span class="text-green-600 text-2xl">"</span>
          </blockquote>

          <!-- Tags décoratifs -->
          <div class="flex flex-wrap gap-2 mt-4">
            <span class="bg-green-100 text-green-800 px-3 py-1 rounded-full text-sm font-medium hover:bg-green-200 transition-colors">
              🌍 Écologie
            </span>
            <span class="bg-blue-100 text-blue-800 px-3 py-1 rounded-full text-sm font-medium hover:bg-blue-200 transition-colors">
              💚 Environnement
            </span>
            <span class="bg-yellow-100 text-yellow-800 px-3 py-1 rounded-full text-sm font-medium hover:bg-yellow-200 transition-colors">
              ♻️ Durable
            </span>
          </div>
        </div>

        <!-- État initial -->
        <div v-else class="text-center py-8 text-gray-500">
          <div class="text-4xl mb-4 animate-bounce">🌿</div>
          <p class="text-lg">Découvrez un fait écologique fascinant !</p>
          <p class="text-sm mt-2 opacity-75">Cliquez sur le bouton de rechargement pour commencer</p>
        </div>
      </div>

      <!-- Effet de brillance -->
      <div class="absolute inset-0 bg-gradient-to-r from-transparent via-white/10 to-transparent transform -skew-x-12 -translate-x-full group-hover:animate-shimmer"></div>
    </div>

    <button class="mt-5 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full text-center cursor-pointer" @click="generateNewFact">
      Générer un nouveau fait
    </button>

    <!-- Compteur de faits -->
    <div v-if="factCount > 0" class="text-center mt-4">
      <span class="text-sm text-gray-600 bg-white/80 backdrop-blur-sm px-4 py-2 rounded-full shadow-sm border border-gray-200 hover:bg-white/90 transition-colors">
        🎯 {{ factCount }} fait{{ factCount > 1 ? 's' : '' }} découvert{{ factCount > 1 ? 's' : '' }}
      </span>
    </div>

    <!-- Indicateur de source -->
    <div class="text-center mt-2">
      <span class="text-xs text-gray-400">
        Généré par IA • Vérifiez toujours les informations importantes
      </span>
    </div>
  </div>
</template>

<style scoped>
@keyframes shimmer {
  0% {
    transform: translateX(-100%) skewX(-12deg);
  }
  100% {
    transform: translateX(200%) skewX(-12deg);
  }
}

@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-shimmer {
  animation: shimmer 2s ease-in-out;
}

.animate-fade-in {
  animation: fade-in 0.6s ease-out;
}

/* Animation pour le bouton de rechargement */
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.animate-spin {
  animation: spin 1s linear infinite;
}

/* Amélioration de l'accessibilité */
button:focus {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}

/* Responsive design */
@media (max-width: 640px) {
  .text-lg {
    font-size: 1rem;
  }

  .text-xl {
    font-size: 1.125rem;
  }
}
</style>