<template>
  <div>
    <h5 class="mb-4">Liste des contacts</h5>
    
    <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
      <div v-for="contact in contacts" :key="contact.id" class="col">
        <div class="card contact-card h-100 shadow-sm">
          <div class="card-body">
            <div class="d-flex justify-content-between align-items-start mb-3">
              <h6 class="card-title mb-0 fw-bold">{{ contact.name }}</h6>
              <div class="dropdown">
                <button class="btn btn-sm btn-outline-secondary dropdown-toggle" 
                        type="button" 
                        data-bs-toggle="dropdown">
                  <i class="bi bi-three-dots-vertical"></i>
                </button>
                <ul class="dropdown-menu dropdown-menu-end">
                  <li>
                    <button class="dropdown-item" @click="$emit('edit-contact', contact)">
                      <i class="bi bi-pencil me-2"></i>Modifier
                    </button>
                  </li>
                  <li>
                    <button class="dropdown-item text-danger" @click="deleteContact(contact.id)">
                      <i class="bi bi-trash me-2"></i>Supprimer
                    </button>
                  </li>
                </ul>
              </div>
            </div>
            
            <div class="contact-details">
              <p class="mb-2">
                <i class="bi bi-envelope me-2 text-primary"></i>
                <a :href="`mailto:${contact.email}`">{{ contact.email }}</a>
              </p>
              <p class="mb-0">
                <i class="bi bi-telephone me-2 text-success"></i>
                <a :href="`tel:${contact.phone}`">{{ contact.phone }}</a>
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ContactListComponent",
  props: {
    contacts: {
      type: Array,
      required: true,
    },
  },
  methods: {
    async deleteContact(id) {
      if (confirm("Voulez-vous vraiment supprimer ce contact ?")) {
        try {
          const response = await fetch(`http://localhost:3000/contacts/${id}`, {
            method: "DELETE",
          });

          if (!response.ok) throw new Error("Échec de la suppression");
          this.$emit("contact-deleted", "Contact supprimé avec succès");
        } catch (error) {
          this.$emit("error", error.message);
        }
      }
    },
  },
};
</script>

<style scoped>
.contact-card {
  border-radius: 10px;
  border: 1px solid rgba(0,0,0,0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.contact-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 5px 15px rgba(0,0,0,0.1) !important;
}

.contact-details i {
  width: 20px;
  text-align: center;
}

.dropdown-toggle::after {
  display: none;
}

.card-title {
  color: #2c3e50;
}

a {
  text-decoration: none;
  color: inherit;
}
</style>