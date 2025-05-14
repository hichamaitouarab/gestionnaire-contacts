<template>
  <div class="container mt-5">
    <h1 class="mb-4">Gestionnaire de Contacts</h1>

    <!-- Notification globale -->
    <div class="position-fixed top-0 end-0 p-3" style="z-index: 11">
      <div v-if="globalNotification.message" 
           :class="`alert alert-${globalNotification.type} alert-dismissible fade show`"
           role="alert">
        {{ globalNotification.message }}
        <button type="button" class="btn-close" @click="globalNotification.message = ''"></button>
      </div>
    </div>

    <button 
  v-if="!showForm && !editingContact"
  @click="showForm = true"
  class="btn btn-primary mb-4 rounded-pill px-4"
>
  <i class="bi bi-plus-circle me-2"></i> Ajouter un contact
</button>

    <ContactForm 
      v-if="showForm || editingContact"
      @contact-updated="handleContactUpdated" 
      :editingContact="editingContact" 
      @cancel-edit="handleCancelEdit"
      @error="showGlobalNotification('danger', $event)"
      :minLoadingDuration="2000"
    />

    <div v-if="loading" class="text-center my-5">
      <div class="spinner-border slow-spinner" style="width: 3rem; height: 3rem;" role="status">
        <span class="visually-hidden">Chargement...</span>
      </div>
      <p class="mt-2">Chargement des contacts...</p>
    </div>

    <ContactList 
      v-else-if="contacts.length > 0"
      :contacts="contacts" 
      @edit-contact="startEdit" 
      @contact-deleted="handleContactDeleted" 
      @error="showGlobalNotification('danger', $event)"
    />

    <div v-else class="alert alert-info">
      <i class="bi bi-info-circle"></i> Aucun contact trouvé. Cliquez sur "Ajouter un contact" pour commencer.
    </div>
  </div>
</template>

<script>
import ContactForm from './components/ContactForm.vue'
import ContactList from './components/ContactList.vue'

export default {
  name: 'ContactManagerApp',
  components: { ContactForm, ContactList },
  data() {
    return {
      contacts: [],
      loading: true,
      editingContact: null,
      showForm: false,
      globalNotification: {
        type: '',
        message: ''
      }
    }
  },
  methods: {
    async refreshContacts() {
      const startTime = Date.now()
      try {
        this.loading = true
        const response = await fetch('http://localhost:3000/contacts')
        if (!response.ok) throw new Error('Erreur lors du chargement des contacts')
        this.contacts = await response.json()
        
        // Garantir un chargement minimum de 2 secondes
        const elapsed = Date.now() - startTime
        if (elapsed < 2000) {
          await new Promise(resolve => setTimeout(resolve, 2000 - elapsed))
        }
      } catch (error) {
        this.showGlobalNotification('danger', error.message)
      } finally {
        this.loading = false
      }
    },
    startEdit(contact) {
      this.editingContact = contact
      this.showForm = true
    },
    handleContactUpdated(message) {
      this.refreshContacts()
      this.showForm = false
      this.editingContact = null
      this.showGlobalNotification('success', message || 'Opération réussie !')
    },
    handleContactDeleted(message) {
      this.refreshContacts()
      this.showGlobalNotification('success', message || 'Contact supprimé avec succès')
    },
    handleCancelEdit() {
      this.showForm = false
      this.editingContact = null
    },
    showGlobalNotification(type, message) {
      this.globalNotification = { type, message }
      setTimeout(() => {
        this.globalNotification.message = ''
      }, 5000)
    }
  },
  async created() {
    await this.refreshContacts()
  }
}
</script>

<style>
.slow-spinner {
  animation-duration: 1.5s; /* Ralenti l'animation du spinner */
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>