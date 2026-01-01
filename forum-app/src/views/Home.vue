<template>
  <div class="container mt-4">
    <!-- Barre de recherche et filtre catégorie -->
    <b-row class="mb-3 align-items-center">
      <b-col cols="12" md="6">
        <b-form-input
          v-model="searchTerm"
          placeholder="Rechercher une discussion..."
          clearable
        />
      </b-col>
      <b-col cols="12" md="6" class="text-md-end mt-2 mt-md-0">
        <b-form-select
          v-model="selectedCategory"
          :options="categoryOptions"
        />
      </b-col>
    </b-row>

    <!-- Bouton création discussion -->
    <b-button variant="primary" class="mb-3" @click="showNewDiscussionForm = !showNewDiscussionForm">
      {{ showNewDiscussionForm ? "Annuler" : "Créer une nouvelle discussion" }}
    </b-button>

    <!-- Formulaire nouvelle discussion -->
    <NewDiscussionForm
      v-if="showNewDiscussionForm"
      @discussion-created="handleDiscussionCreated"
    />

    <!-- Onglets Discussions -->
    <b-tabs card>
      <b-tab title="Discussions récentes" active>
        <DiscussionList :discussions="filteredDiscussionsRecent" />
      </b-tab>
      <b-tab title="Discussions populaires">
        <DiscussionList :discussions="filteredDiscussionsPopular" />
      </b-tab>
    </b-tabs>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import DiscussionList from "../components/DiscussionList.vue";
import NewDiscussionForm from "../components/NewDiscussionForm.vue";
import { collection, query, orderBy, onSnapshot } from "firebase/firestore";
import { db } from "../firebase";

// Données
const discussions = ref([]);
const searchTerm = ref("");
const selectedCategory = ref("Toutes les catégories");
const showNewDiscussionForm = ref(false);

// Catégories
const categories = [
  { value: "Toutes les catégories", text: "Toutes les catégories" },
  { value: "Général", text: "Général" },
  { value: "Technologie", text: "Technologie" },
  { value: "Actualité", text: "Actualité" },
  { value: "Autre", text: "Autre" }
];
const categoryOptions = categories;

// Charger discussions depuis Firestore
const loadDiscussions = () => {
  const q = query(collection(db, "discussions"), orderBy("date de création", "desc"));
  onSnapshot(q, snap => {
    discussions.value = snap.docs.map(doc => ({
      id: doc.id,
      ...doc.data(),
      popularity: doc.data().popularity || 0 // si tu veux gérer popularité
    }));
  });
};

// Filtrage + recherche
const filteredDiscussionsRecent = computed(() => {
  const term = searchTerm.value.toLowerCase();
  return discussions.value
    .filter(d => selectedCategory.value === "Toutes les catégories" || d.catégorie === selectedCategory.value)
    .filter(d => d.titre.toLowerCase().includes(term) || d.contenu.toLowerCase().includes(term))
    .slice(0, 10);
});

const filteredDiscussionsPopular = computed(() => {
  const term = searchTerm.value.toLowerCase();
  return discussions.value
    .filter(d => selectedCategory.value === "Toutes les catégories" || d.catégorie === selectedCategory.value)
    .filter(d => d.titre.toLowerCase().includes(term) || d.contenu.toLowerCase().includes(term))
    .sort((a, b) => b.popularity - a.popularity)
    .slice(0, 10);
});

// Gestion formulaire nouvelle discussion
function handleDiscussionCreated() {
  showNewDiscussionForm.value = false;
}

// Lifecycle
onMounted(() => {
  loadDiscussions();
});
</script>

<style scoped>
.container {
  max-width: 900px;
}
</style>
