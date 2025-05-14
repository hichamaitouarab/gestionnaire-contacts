<template>
  <div class="card mb-4 shadow-sm">
    <div class="card-body">
      <h5 class="card-title d-flex justify-content-between align-items-center">
        <span>{{ isEditing ? "Modifier contact" : "Nouveau contact" }}</span>
        <button @click="cancel" class="btn-close"></button>
      </h5>

      <div
        v-if="status.message"
        :class="`alert alert-${status.type} alert-dismissible fade show`"
      >
        {{ status.message }}
        <button
          type="button"
          class="btn-close"
          @click="status.message = ''"
        ></button>
      </div>

      <form @submit.prevent="handleSubmit">
        <div class="mb-3">
          <label class="form-label"
            >Nom complet <span class="text-danger">*</span></label
          >
          <input
            v-model.trim="form.name"
            class="form-control"
            :class="{ 'is-invalid': errors.name }"
            required
          />
          <div v-if="errors.name" class="invalid-feedback">
            {{ errors.name }}
          </div>
        </div>

        <div class="mb-3">
          <label class="form-label"
            >Email <span class="text-danger">*</span></label
          >
          <input
            v-model.trim="form.email"
            type="email"
            class="form-control"
            :class="{ 'is-invalid': errors.email }"
            required
          />
          <div v-if="errors.email" class="invalid-feedback">
            {{ errors.email }}
          </div>
        </div>

        <div class="mb-3">
          <label class="form-label"
            >Téléphone <span class="text-danger">*</span></label
          >
          <input
            v-model.trim="form.phone"
            class="form-control"
            :class="{ 'is-invalid': errors.phone }"
            required
          />
          <div v-if="errors.phone" class="invalid-feedback">
            {{ errors.phone }}
          </div>
        </div>

        <div class="d-grid gap-2 d-md-flex justify-content-md-end">
          <button
            @click="cancel"
            type="button"
            class="btn btn-outline-secondary me-md-2"
            :disabled="isSubmitting"
          >
            Annuler
          </button>

          <button
            type="submit"
            class="btn btn-primary"
            :disabled="isSubmitting"
          >
            <template v-if="!isSubmitting">
              <i
                class="bi"
                :class="isEditing ? 'bi-check-circle' : 'bi-plus-circle'"
              ></i>
              {{ isEditing ? "Mettre à jour" : "Ajouter" }}
            </template>
            <template v-else>
              <span
                class="spinner-border spinner-border-sm slow-spinner"
                role="status"
                aria-hidden="true"
              ></span>
              {{ isEditing ? "Enregistrement..." : "Ajout en cours..." }}
            </template>
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: "ContactFormComponent",
  props: {
    editingContact: {
      type: Object,
      default: null,
    },
    minLoadingDuration: {
      type: Number,
      default: 2000, // 2 secondes par défaut
    },
  },
  data() {
    return {
      form: { name: "", email: "", phone: "" },
      isEditing: false,
      isSubmitting: false,
      errors: {
        name: "",
        email: "",
        phone: "",
      },
      status: {
        type: "",
        message: "",
      },
    };
  },
  watch: {
    editingContact: {
      handler(newVal) {
        if (newVal) {
          this.form = JSON.parse(JSON.stringify(newVal));
          this.isEditing = true;
        } else {
          this.isEditing = false;
        }
      },
      immediate: true,
    },
  },
  methods: {
    validateForm() {
      let isValid = true;
      this.errors = { name: "", email: "", phone: "" };

      if (!this.form.name) {
        this.errors.name = "Le nom est requis";
        isValid = false;
      }

      if (!this.form.email) {
        this.errors.email = "L'email est requis";
        isValid = false;
      } else if (!/^\S+@\S+\.\S+$/.test(this.form.email)) {
        this.errors.email = "Veuillez entrer un email valide";
        isValid = false;
      }

      if (!this.form.phone) {
        this.errors.phone = "Le téléphone est requis";
        isValid = false;
      }

      return isValid;
    },
    async handleSubmit() {
      if (!this.validateForm()) return;

      const startTime = Date.now();
      this.isSubmitting = true;
      this.status = { type: "", message: "" };

      try {
        const url = this.isEditing
          ? `http://localhost:3000/contacts/${this.form.id}`
          : "http://localhost:3000/contacts";

        const method = this.isEditing ? "PUT" : "POST";

        const response = await fetch(url, {
          method,
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.form),
        });

        if (!response.ok)
          throw new Error(
            this.isEditing ? "Échec de la mise à jour" : "Échec de l'ajout"
          );

        const successMessage = this.isEditing
          ? "Contact mis à jour avec succès !"
          : "Contact ajouté avec succès !";

        this.status = {
          type: "success",
          message: successMessage,
        };

        // Garantir un temps minimum de chargement
        const elapsed = Date.now() - startTime;
        const remainingTime = Math.max(0, this.minLoadingDuration - elapsed);
        await new Promise((resolve) => setTimeout(resolve, remainingTime));

        this.$emit("contact-updated", successMessage);
        this.resetForm();
      } catch (error) {
        this.status = {
          type: "danger",
          message: error.message || "Une erreur est survenue",
        };
        this.$emit("error", error.message);
      } finally {
        this.isSubmitting = false;
      }
    },
    resetForm() {
      this.form = { name: "", email: "", phone: "" };
      this.errors = { name: "", email: "", phone: "" };
    },
    cancel() {
      this.resetForm();
      this.$emit("cancel-edit");
    },
  },
};
</script>

<style scoped>
.slow-spinner {
  animation-duration: 1.5s;
}

.card {
  transition: all 0.3s ease;
}
.card-title .btn-close {
  font-size: 0.75rem;
}
.invalid-feedback {
  display: block;
}
.btn-primary {
  background-color: #4361ee;
  border: none;
  padding: 8px 20px;
  border-radius: 8px;
  transition: all 0.3s;
}

.btn-primary:hover {
  background-color: #3a56d4;
  transform: translateY(-1px);
}

.btn-outline-secondary {
  border-radius: 8px;
  transition: all 0.3s;
}

.btn-outline-secondary:hover {
  background-color: #f8f9fa;
}
</style>
