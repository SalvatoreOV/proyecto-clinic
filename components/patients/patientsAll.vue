<template>
  <v-container>
    <v-row align="center" justify="end">
      <div>
        <span class="title">
          Lab Appointments
        </span>
      </div>
      <v-spacer />
      <v-text-field
        label="Search"
        append-icon="mdi-magnify"
        single-line
        hide-details
        outlined
        class="mr-3"
      />
      <v-btn class="btnFilter">
        <span>
          Filter
        </span>
        <v-icon>
          mdi-filter-outline
        </v-icon>
      </v-btn>
      <v-btn class="btnPatient" @click="dialogAddPacient = true">
        <v-icon>
          mdi-plus
        </v-icon>
        <span>
          Add Patient
        </span>
      </v-btn>
    </v-row>
    <v-row align="center" justify="start">
      <v-data-table
        :headers="headers"
        :items="pacientes"
        :items-per-page="10"
        dense
        class="mt-4"
      >
        <template #[`item.photo`]="{ item }">
          <v-avatar size="40">
            <v-img
              v-if="item.photo"
              :src="'data:image/jpeg;base64,' + item.photo"
              :alt="item.fullName"
            />
            <v-icon v-else>mdi-account-circle</v-icon>
          </v-avatar>
        </template>
        <template #[`item.actions`]="{ item }">
          <v-icon small color="primary" title="Editar" @click="editPatient(item)">
            mdi-pencil
          </v-icon>
          <v-icon small color="red" title="Borrar" @click="deletePatient(item)">
            mdi-delete
          </v-icon>
        </template>
      </v-data-table>
    </v-row>
    <v-dialog v-model="dialogAddPacient" persistent max-width="800px">
      <v-card color="indigo lighten-5" elevation="0">
        <v-card-title>
          Add Patient
          <v-spacer />
          <v-btn icon class="mr-2" color="grey darken-1" @click="dialogAddPacient = false">
            <v-icon>
              mdi-close
            </v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-tabs v-model="activeTab">
            <v-tab>Demographics</v-tab>
            <v-tab>Adress</v-tab>
            <v-tab>Photo</v-tab>
          </v-tabs>

          <v-tabs-items v-model="activeTab">
            <!-- Tab Demographics -->
            <v-tab-item>
              <v-form style="margin: 10px !important;">
                <v-row>
                  <v-col cols="12">
                    <v-text-field v-model="paciente.fullName" label="Full Name" required outlined />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.email" label="Email" required outlined />
                  </v-col>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.mobile" label="Mobile" required outlined />
                  </v-col>
                  <v-col cols="4">
                    <v-menu ref="menu" :close-on-content-click="false" transition="scale-transition" offset-y min-width="auto">
                      <template #activator="{ on, attrs }">
                        <v-text-field
                          v-model="paciente.dateBirth"
                          label="Date of Birth"
                          prepend-inner-icon="mdi-calendar"
                          readonly
                          outlined
                          v-bind="attrs"
                          v-on="on"
                        />
                      </template>
                      <v-date-picker v-model="paciente.dateBirth" @input="menu = false" />
                    </v-menu>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.motherTongue" label="Mother Tongue" required outlined />
                  </v-col>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.govId" label="Goberment ID" required outlined />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.sex" label="sex" required outlined />
                  </v-col>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.maritalStatus" label="Marital Status" required outlined />
                  </v-col>
                </v-row>
              </v-form>
            </v-tab-item>

            <!-- Address -->
            <v-tab-item>
              <v-form style="margin: 10px !important;">
                <v-row>
                  <v-col cols="12">
                    <v-text-field v-model="paciente.address" label="Address" required outlined />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.city" label="City" required outlined />
                  </v-col>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.zipCode" label="Zip Code" required outlined />
                  </v-col>
                  <v-col cols="4">
                    <v-text-field v-model="paciente.state" label="State" required outlined />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.country" label="Country" required outlined />
                  </v-col>
                  <v-col cols="6">
                    <v-text-field v-model="paciente.district" label="District" required outlined />
                  </v-col>
                </v-row>
              </v-form>
            </v-tab-item>

            <!-- Photo -->
            <v-tab-item>
              <v-form style="margin: 10px !important;">
                <v-row>
                  <v-col cols="12">
                    <v-file-input
                      v-model="photoFile"
                      label="Upload Photo"
                      accept="image/*"
                      outlined
                      prepend-inner-icon="mdi-camera"
                      @change="handleFileUpload"
                    />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12">
                    <v-img v-if="paciente.photo" :src="'data:image/jpeg;base64,' + paciente.photo" max-width="200" max-height="200" class="mt-2" />
                  </v-col>
                </v-row>
              </v-form>
            </v-tab-item>
          </v-tabs-items>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn class="btnPatient" @click="savePatient">
            <span>Add Patient</span>
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  data () {
    return {
      pacientes: [],
      headers: [
        { text: 'Photo', value: 'photo', sortable: false },
        { text: 'Full Name', value: 'fullName' },
        { text: 'Email', value: 'email' },
        { text: 'Mobile', value: 'mobile' },
        { text: 'Date Birth', value: 'dateBirth' },
        { text: 'Mother Tongue', value: 'motherTongue' },
        { text: 'Government ID', value: 'govId' },
        { text: 'Actions', value: 'actions', sortable: false }
      ],
      dialogAddPacient: false,
      paciente: {
        fullName: '',
        email: '',
        mobile: '',
        dateBirth: '',
        motherTongue: '',
        govId: '',
        sex: '',
        maritalStatus: '',
        address: '',
        city: '',
        zipCode: '',
        state: '',
        country: '',
        district: '',
        international: 'false',
        photo: ''
      },
      photoFile: null,
      activeTab: 0,
      menu: false,
      isValidImage: false
    }
  },
  mounted () {
    this.loadPacientes()
  },
  methods: {
    validateFileSize (file) {
      const maxSize = 500 * 1024 // 500KB in bytes
      if (file.size > maxSize) {
        this.$store.dispatch('alert/triggerAlert', {
          message: 'La imagen debe ser menor a 500KB',
          type: 'error'
        })
        this.isValidImage = false
        return false
      }
      this.isValidImage = true
      return true
    },
    handleFileUpload (event) {
      const file = event
      if (!file) {
        this.isValidImage = false
        this.paciente.photo = ''
        return
      }
      if (!this.validateFileSize(file)) {
        this.photoFile = null
        this.paciente.photo = ''
        return
      }
      const reader = new FileReader()
      reader.onload = (e) => {
        this.paciente.photo = e.target.result.split(',')[1]
      }
      reader.readAsDataURL(file)
    },
    async loadPacientes () {
      try {
        const response = await this.$axios.get('/pacientes')
        console.log('Response from API:', response.data)
        if (response.data && response.data.pacientes) {
          this.pacientes = response.data.pacientes
        } else if (response.data) {
          // Si los datos vienen directamente en data
          this.pacientes = response.data
        } else {
          throw new Error('No se recibieron datos de pacientes')
        }
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los pacientes'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async savePatient () {
      if (!this.isValidImage || !this.paciente.photo) {
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Por favor sube una imagen válida (máximo 500KB)',
          type: 'error'
        })
        return
      }
      try {
        await this.$axios.post('/pacientes/create', this.paciente)
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Paciente Guardado Satisfactoriamente',
          type: 'success'
        })
        this.dialogAddPacient = false
        this.loadPacientes()
      } catch (error) {
        const errorMessage = error.message || 'Error al registrar los usuarios'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    editPatient (patient) {
      this.paciente = { ...patient }
      this.dialogAddPacient = true
    },
    async deletePatient (patient) {
      try {
        await this.$axios.delete(`/pacientes/delete/${patient.id}`)
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Paciente Eliminado Con Éxito',
          type: 'success'
        })
        this.loadPacientes()
      } catch (error) {
        const errorMessage = error.message || 'Error al eliminar el paciente'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    }
  }
}

</script>

<style scoped>
.title{
  font-size: 24px;
  font-weight: 500;
  color: #101828;
}

.btnFilter {
  background-color: white;
  color: #101828 !important;
  margin-right: 10px !important;
  border: 1px solid #101828 !important;
  border-radius: 5px !important;
  padding: 5px 10px !important;
  font-size: 14px !important;
  text-transform: none !important;
}

.btnPatient {
  text-transform: none !important;
  color: white;
  border-radius: 5px !important;
  padding: 5px 10px !important;
  background-color: #0e1680 !important;
  font-size: 14px !important;
  margin-right: 10px !important;
  font-weight: 500 !important;
}

</style>
