<template>
  <v-container>
    <v-row align="center" justify="end">
      <div>
        <span class="title"> Patients List </span>
      </div>
      <v-spacer />
      <v-text-field
        v-model="searchQuery"
        label="Search"
        append-icon="mdi-magnify"
        single-line
        hide-details
        outlined
        dense
        class="mr-3 search-field"
      />
      <v-btn class="btnFilter">
        <span> Filter </span>
        <v-icon right>
          mdi-filter-outline
        </v-icon>
      </v-btn>
      <v-btn class="btnPatient" @click="openAddPatientDialog">
        <v-icon left>
          mdi-plus
        </v-icon>
        <span> Add Patient </span>
      </v-btn>
      <v-btn icon class="ml-2">
        <v-icon>mdi-dots-vertical</v-icon>
      </v-btn>
    </v-row>
    <v-row align="center" justify="start">
      <v-data-table
        v-model="selected"
        :headers="headers"
        :items="filteredPacientes"
        :items-per-page="10"
        item-key="id"
        show-select
        class="mt-4 patients-table elevation-1"
        hide-default-footer
      >
        <template #[`item.patientName`]="{ item }">
          <div class="d-flex align-center">
            <v-avatar size="32" class="mr-3">
              <v-img
                v-if="item.photo"
                :src="'data:image/jpeg;base64,' + item.photo"
                :alt="item.fullName"
              />
              <v-icon v-else size="32">
                mdi-account-circle
              </v-icon>
            </v-avatar>
            <div>
              <div class="font-weight-medium text-truncate">
                {{ item.fullName }}
              </div>
              <div class="text-caption grey--text text-truncate">
                {{ item.email }}
              </div>
            </div>
          </div>
        </template>
        <template #[`item.date`]="{ item }">
          <span class="date-display">{{ formatDate(item.date) }}</span>
        </template>
        <template #[`item.patientId`]="{ item }">
          <span class="text-truncate">{{ item.patientId }}</span>
        </template>
        <template #[`item.sex`]="{ item }">
          <span class="text-truncate">{{ item.sex }}</span>
        </template>
        <template #[`item.age`]="{ item }">
          <span class="text-truncate">{{ item.age }}</span>
        </template>
        <template #[`item.diseases`]="{ item }">
          <span class="text-truncate">{{ item.diseases }}</span>
        </template>
        <template #[`item.doctorName`]="{ item }">
          <span class="text-truncate">{{ item.doctorName }}</span>
        </template>
        <template #[`item.status`]="{ item }">
          <v-chip
            small
            :color="getStatusColor(item.status)"
            :text-color="getStatusChipTextColor(item.status)"
            :class="getStatusChipClasses(item.status)"
            label
          >
            <v-icon left small class="mr-1">
              mdi-circle
            </v-icon>
            {{ item.status }}
          </v-chip>
        </template>
        <template #[`item.actions`]="{ item }">
          <v-btn
            icon
            small
            color="grey darken-1"
            class="mr-1"
            title="Borrar"
            @click="deletePatient(item)"
          >
            <v-icon small>
              mdi-delete-outline
            </v-icon>
          </v-btn>
          <v-btn icon small color="grey darken-1" title="Editar" @click="editPatient(item)">
            <v-icon small>
              mdi-pencil-outline
            </v-icon>
          </v-btn>
        </template>
        <template #footer>
          <v-divider />
          <div class="d-flex justify-space-between align-center pa-4">
            <v-btn text small>
              <v-icon left>
                mdi-arrow-left
              </v-icon>
              Previous
            </v-btn>
            <div>
              <v-btn text small class="mx-1" :class="{'active-page': currentPage === 1}" @click="currentPage = 1">
                1
              </v-btn>
              <v-btn text small class="mx-1" :class="{'active-page': currentPage === 2}" @click="currentPage = 2">
                2
              </v-btn>
              <v-btn text small class="mx-1" :class="{'active-page': currentPage === 3}" @click="currentPage = 3">
                3
              </v-btn>
              <span class="mx-1">...</span>
              <v-btn text small class="mx-1" :class="{'active-page': currentPage === 8}" @click="currentPage = 8">
                8
              </v-btn>
              <v-btn text small class="mx-1" :class="{'active-page': currentPage === 9}" @click="currentPage = 9">
                9
              </v-btn>
              <v-btn text small class="mx-1" :class="{'active-page': currentPage === 10}" @click="currentPage = 10">
                10
              </v-btn>
            </div>
            <v-btn text small>
              Next
              <v-icon right>
                mdi-arrow-right
              </v-icon>
            </v-btn>
          </div>
        </template>
      </v-data-table>
    </v-row>
    <!-- Add Patient Dialog -->
    <v-dialog v-model="dialogAddPacient" persistent max-width="800px">
      <v-card color="white" elevation="2">
        <v-card-title class="primary-header d-flex align-center">
          <span class="font-weight-bold">Add Appointment</span>
          <v-spacer />
          <v-btn
            icon
            class="mr-2"
            color="grey darken-1"
            @click="dialogAddPacient = false"
          >
            <v-icon>
              mdi-close
            </v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text class="pa-0">
          <div class="tab-container">
            <v-tabs
              v-model="activeTab"
              background-color="#f8f9fc"
              grow
            >
              <v-tab class="text-none py-4 tab-button" :class="{'active-tab': activeTab === 0}">
                Demographics
              </v-tab>
              <v-tab class="text-none py-4 tab-button" :class="{'active-tab': activeTab === 1}">
                Address
              </v-tab>
              <v-tab class="text-none py-4 tab-button" :class="{'active-tab': activeTab === 2}">
                Photo
              </v-tab>
            </v-tabs>
          </div>

          <v-tabs-items v-model="activeTab">
            <!-- Tab 1: Demographics -->
            <v-tab-item>
              <div class="tab-content-container pa-6">
                <div class="form-field-container mb-4">
                  <div class="form-label">
                    Full Name
                  </div>
                  <v-text-field
                    v-model="paciente.fullName"
                    placeholder="Enter your full name"
                    outlined
                    dense
                    hide-details="auto"
                    class="form-field"
                  />
                </div>

                <div class="row-container">
                  <div class="form-field-container mb-4 mr-4">
                    <div class="form-label">
                      Email ID
                    </div>
                    <v-text-field
                      v-model="paciente.email"
                      placeholder="Enter your e-mail"
                      outlined
                      dense
                      hide-details="auto"
                      class="form-field"
                    />
                  </div>

                  <div class="form-field-container mb-4">
                    <div class="form-label">
                      Mobile Number
                    </div>
                    <v-text-field
                      v-model="paciente.mobile"
                      placeholder="Enter your mobile number"
                      outlined
                      dense
                      hide-details="auto"
                      class="form-field"
                    />
                  </div>
                </div>

                <div class="row-container">
                  <div class="form-field-container mb-4 mr-4">
                    <div class="form-label">
                      Date of Birth
                    </div>
                    <v-menu
                      ref="menu"
                      :close-on-content-click="false"
                      transition="scale-transition"
                      offset-y
                      min-width="auto"
                    >
                      <template #activator="{ on, attrs }">
                        <v-text-field
                          v-model="paciente.dateBirth"
                          placeholder="Enter date"
                          prepend-inner-icon="mdi-calendar"
                          readonly
                          outlined
                          dense
                          hide-details="auto"
                          class="form-field"
                          v-bind="attrs"
                          v-on="on"
                        />
                      </template>
                      <v-date-picker
                        v-model="paciente.dateBirth"
                        @input="menu = false"
                      />
                    </v-menu>
                  </div>

                  <div class="form-field-container mb-4 mr-4">
                    <div class="form-label">
                      Mother Tongue
                    </div>
                    <v-text-field
                      v-model="paciente.motherTongue"
                      placeholder="Enter your mother tongue"
                      outlined
                      dense
                      hide-details="auto"
                      class="form-field"
                    />
                  </div>
                </div>

                <div class="row-container">
                  <div class="form-field-container mb-4">
                    <div class="form-label">
                      Government ID
                    </div>
                    <v-text-field
                      v-model="paciente.govId"
                      placeholder="Enter govt id"
                      outlined
                      dense
                      hide-details="auto"
                      class="form-field"
                    />
                  </div>
                </div>

                <div class="row-container">
                  <div class="form-field-container mb-4 mr-4">
                    <div class="form-label">
                      Sex
                    </div>
                    <div class="radio-group-container">
                      <v-radio-group
                        v-model="paciente.sex"
                        row
                        hide-details
                      >
                        <v-radio
                          label="Male"
                          value="Male"
                          color="primary"
                        />
                        <v-radio
                          label="Female"
                          value="Female"
                          color="primary"
                        />
                      </v-radio-group>
                    </div>
                  </div>

                  <div class="form-field-container mb-4">
                    <div class="form-label">
                      Marital Status
                    </div>
                    <div class="radio-group-container">
                      <v-radio-group
                        v-model="paciente.maritalStatus"
                        row
                        hide-details
                      >
                        <v-radio
                          label="Single"
                          value="Single"
                          color="primary"
                        />
                        <v-radio
                          label="Married"
                          value="Married"
                          color="primary"
                        />
                        <v-radio
                          label="Widow"
                          value="Widow"
                          color="primary"
                        />
                      </v-radio-group>
                    </div>
                  </div>
                </div>
              </div>
            </v-tab-item>

            <!-- Tab 2: Address -->
            <v-tab-item>
              <div class="tab-content-container pa-6">
                <div class="form-field-container mb-4">
                  <div class="form-label">
                    Address
                  </div>
                  <v-text-field
                    v-model="paciente.address"
                    placeholder="Enter your home address"
                    outlined
                    dense
                    hide-details="auto"
                    class="form-field"
                  />
                </div>

                <div class="row-container">
                  <div class="form-field-container mb-4 mr-4">
                    <div class="form-label">
                      City
                    </div>
                    <v-text-field
                      v-model="paciente.city"
                      placeholder="Enter your city name"
                      outlined
                      dense
                      hide-details="auto"
                      class="form-field"
                    />
                  </div>
                  <div class="form-field-container mb-4">
                    <div class="form-label">
                      Pin Code
                    </div>
                    <v-text-field
                      v-model="paciente.zipCode"
                      placeholder="Enter pin"
                      outlined
                      dense
                      hide-details="auto"
                      class="form-field"
                    />
                  </div>
                </div>

                <div class="row-container">
                  <div class="form-field-container mb-4 mr-4">
                    <div class="form-label">
                      State
                    </div>
                    <v-text-field
                      v-model="paciente.state"
                      placeholder="Select your state"
                      outlined
                      dense
                      hide-details="auto"
                      class="form-field"
                    />
                  </div>
                  <div class="form-field-container mb-4">
                    <div class="form-label">
                      Country
                    </div>
                    <v-text-field
                      v-model="paciente.country"
                      placeholder="Select your country"
                      outlined
                      dense
                      hide-details="auto"
                      class="form-field"
                    />
                  </div>
                </div>

                <div class="form-field-container mb-4">
                  <div class="form-label">
                    District
                  </div>
                  <v-text-field
                    v-model="paciente.district"
                    placeholder="Select your district"
                    outlined
                    dense
                    hide-details="auto"
                    class="form-field"
                  />
                </div>

                <div class="checkbox-container mt-4">
                  <v-checkbox
                    v-model="paciente.international"
                    label="International Patient"
                    color="primary"
                    hide-details="auto"
                  />
                </div>
              </div>
            </v-tab-item>

            <!-- Tab 3: Photo -->
            <v-tab-item>
              <div class="tab-content-container pa-6">
                <div class="form-field-container mb-4">
                  <div class="form-label mb-2">
                    Upload Photo
                  </div>
                  <v-file-input
                    v-model="photoFile"
                    placeholder="Click to upload or drag and drop"
                    accept="image/*"
                    outlined
                    dense
                    prepend-inner-icon="mdi-camera"
                    hide-details="auto"
                    class="form-field"
                    :error-messages="photoRequired ? 'Photo is required' : ''"
                    @change="handleFileUpload"
                  />
                </div>

                <div class="d-flex justify-center my-6">
                  <v-avatar size="150" class="elevation-1">
                    <v-img
                      v-if="paciente.photo"
                      :src="'data:image/jpeg;base64,' + paciente.photo"
                    />
                    <v-icon v-else size="150" color="grey lighten-1">
                      mdi-account
                    </v-icon>
                  </v-avatar>
                </div>

                <div class="text-center grey--text text--darken-1">
                  <small>Upload a clear photo (max 500KB)</small>
                </div>
              </div>
            </v-tab-item>

            <!-- Tab 4: Details -->
            <v-tab-item>
              <div class="tab-content-container pa-6">
                <div class="text-center grey--text my-6">
                  Additional details can be added here
                </div>
              </div>
            </v-tab-item>

            <!-- Tab 5: Notes -->
            <v-tab-item>
              <div class="tab-content-container pa-6">
                <div class="text-center grey--text my-6">
                  Notes and additional information can be added here
                </div>
              </div>
            </v-tab-item>
          </v-tabs-items>
        </v-card-text>

        <v-card-actions class="pa-6 action-footer">
          <v-btn v-if="activeTab > 0" text color="primary" class="back-button" @click="activeTab--">
            <v-icon left>
              mdi-arrow-left
            </v-icon>
            Back to Referrer
          </v-btn>
          <v-spacer />
          <v-btn text class="cancel-button mr-3" @click="dialogAddPacient = false">
            Cancel
          </v-btn>
          <v-btn v-if="activeTab < 2" color="primary" class="next-button" @click="nextTab">
            Next
            <v-icon right>
              mdi-arrow-right
            </v-icon>
          </v-btn>
          <v-btn v-else color="primary" class="action-button" @click="validateAndSave">
            <v-icon right>
              mdi-arrow-right
            </v-icon>
            Add Patient
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit Patient Dialog -->
    <v-dialog v-model="dialogEditPacient" persistent max-width="800px">
      <v-card color="white" elevation="2">
        <v-card-title class="primary-header">
          <div class="d-flex align-center">
            <v-btn icon class="mr-2" @click="dialogEditPacient = false">
              <v-icon>mdi-arrow-left</v-icon>
            </v-btn>
            <span>Patient Details</span>
          </div>
          <v-spacer />
          <v-btn
            icon
            class="mr-2"
            color="grey darken-1"
            @click="dialogEditPacient = false"
          >
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>

        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12" class="pb-0">
                <div class="d-flex">
                  <v-avatar size="64" class="mr-4">
                    <v-img
                      v-if="paciente.photo"
                      :src="'data:image/jpeg;base64,' + paciente.photo"
                      :alt="paciente.fullName"
                    />
                    <v-icon v-else size="64" color="grey lighten-1">
                      mdi-account-circle
                    </v-icon>
                  </v-avatar>
                  <div>
                    <div class="text-h6 font-weight-bold">
                      {{ paciente.fullName }}
                    </div>
                    <div class="text-body-1 grey--text">
                      {{ calculateAge(paciente.dateBirth) }} years, {{ paciente.sex }}
                    </div>
                  </div>
                </div>
              </v-col>
            </v-row>

            <v-divider class="my-4" />

            <v-row>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.fullName"
                  label="Patient Name"
                  outlined
                  dense
                />
              </v-col>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.email"
                  label="Email"
                  outlined
                  dense
                />
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.mobile"
                  label="Phone"
                  outlined
                  dense
                />
              </v-col>
              <v-col cols="12" md="6">
                <v-menu
                  ref="menuEdit"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template #activator="{ on, attrs }">
                    <v-text-field
                      v-model="paciente.dateBirth"
                      label="Date of Birth"
                      prepend-inner-icon="mdi-calendar"
                      readonly
                      outlined
                      dense
                      v-bind="attrs"
                      v-on="on"
                    />
                  </template>
                  <v-date-picker
                    v-model="paciente.dateBirth"
                    @input="$refs.menuEdit.save(paciente.dateBirth)"
                  />
                </v-menu>
              </v-col>
            </v-row>

            <v-row>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.govId"
                  label="Government ID"
                  outlined
                  dense
                />
              </v-col>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.motherTongue"
                  label="Mother Tongue"
                  outlined
                  dense
                />
              </v-col>
            </v-row>

            <v-row>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.sex"
                  label="Sex"
                  outlined
                  dense
                />
              </v-col>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.maritalStatus"
                  label="Marital Status"
                  outlined
                  dense
                />
              </v-col>
            </v-row>

            <v-divider class="my-4" />
            <v-subheader class="pl-0 font-weight-medium">
              Address
            </v-subheader>

            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="paciente.address"
                  label="Address"
                  outlined
                  dense
                />
              </v-col>
            </v-row>

            <v-row>
              <v-col cols="12" md="4">
                <v-text-field
                  v-model="paciente.city"
                  label="City"
                  outlined
                  dense
                />
              </v-col>
              <v-col cols="12" md="4">
                <v-text-field
                  v-model="paciente.state"
                  label="State"
                  outlined
                  dense
                />
              </v-col>
              <v-col cols="12" md="4">
                <v-text-field
                  v-model="paciente.zipCode"
                  label="Zip Code"
                  outlined
                  dense
                />
              </v-col>
            </v-row>

            <v-row>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.country"
                  label="Country"
                  outlined
                  dense
                />
              </v-col>
              <v-col cols="12" md="6">
                <v-text-field
                  v-model="paciente.district"
                  label="District"
                  outlined
                  dense
                />
              </v-col>
            </v-row>

            <v-row>
              <v-col cols="12">
                <v-checkbox
                  v-model="paciente.international"
                  label="International Patient"
                  color="primary"
                />
              </v-col>
            </v-row>

            <v-divider class="my-4" />
            <v-subheader class="pl-0 font-weight-medium">
              Photo
            </v-subheader>

            <v-row>
              <v-col cols="12" md="6">
                <v-file-input
                  v-model="photoFile"
                  label="Upload Photo"
                  accept="image/*"
                  outlined
                  dense
                  prepend-inner-icon="mdi-camera"
                  @change="handleFileUpload"
                />
              </v-col>
              <v-col cols="12" md="6" class="d-flex align-center justify-center">
                <v-avatar size="100" class="elevation-1">
                  <v-img
                    v-if="paciente.photo"
                    :src="'data:image/jpeg;base64,' + paciente.photo"
                  />
                  <v-icon v-else size="100" color="grey lighten-1">
                    mdi-account
                  </v-icon>
                </v-avatar>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-card-actions class="pa-4 action-buttons">
          <v-btn text color="red darken-1" @click="confirmDeletePatient">
            <v-icon left>
              mdi-delete
            </v-icon>
            Delete
          </v-btn>
          <v-spacer />
          <v-btn text color="grey darken-1" @click="dialogEditPacient = false">
            Cancel
          </v-btn>
          <v-btn class="btnPatient" @click="updatePatient">
            Save Changes
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Confirm Delete Dialog -->
    <v-dialog v-model="confirmDeleteDialog" max-width="400px">
      <v-card>
        <v-card-title class="text-h6">
          Confirm Deletion
        </v-card-title>
        <v-card-text>
          Are you sure you want to delete this patient? This action cannot be undone.
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text color="grey darken-1" @click="confirmDeleteDialog = false">
            Cancel
          </v-btn>
          <v-btn color="red darken-1" text @click="executeDelete">
            Delete
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
      selected: [],
      pacientes: [],
      searchQuery: '',
      currentPage: 1,
      headers: [
        { text: 'PATIENTS NAME', value: 'patientName', sortable: true, class: 'header-style', cellClass: 'cell-style' },
        { text: 'PATIENT ID', value: 'patientId', sortable: false, class: 'header-style', cellClass: 'cell-style' },
        { text: 'DATE', value: 'date', sortable: true, class: 'header-style', cellClass: 'cell-style' },
        { text: 'SEX', value: 'sex', sortable: true, class: 'header-style', cellClass: 'cell-style' },
        { text: 'AGE', value: 'age', sortable: true, class: 'header-style', cellClass: 'cell-style' },
        { text: 'DISEASES', value: 'diseases', sortable: false, class: 'header-style', cellClass: 'cell-style' },
        { text: 'STATUS', value: 'status', sortable: true, class: 'header-style', cellClass: 'cell-style' },
        { text: 'DOCTOR NAME', value: 'doctorName', sortable: false, class: 'header-style', cellClass: 'cell-style' },
        { text: '', value: 'actions', sortable: false, align: 'end', class: 'header-style', cellClass: 'cell-style' }
      ],
      dialogAddPacient: false,
      dialogEditPacient: false,
      confirmDeleteDialog: false,
      patientToDelete: null,
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
        international: false,
        photo: ''
      },
      photoFile: null,
      activeTab: 0,
      menu: false,
      isValidImage: false,
      formErrors: {
        demographics: false,
        address: false,
        photo: false
      },
      photoRequired: false
    }
  },
  computed: {
    filteredPacientes () {
      if (!this.searchQuery || this.searchQuery.trim() === '') {
        return this.pacientes
      }

      const searchTerm = this.searchQuery.toLowerCase().trim()
      return this.pacientes.filter((patient) => {
        return patient.email.toLowerCase().includes(searchTerm)
      })
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
        let rawPacientes = []
        if (response.data && response.data.pacientes) {
          rawPacientes = response.data.pacientes
        } else if (response.data && Array.isArray(response.data)) {
          rawPacientes = response.data
        } else {
          throw new Error('No se recibieron datos de pacientes válidos')
        }

        this.pacientes = rawPacientes.map((p, index) => ({
          ...p,
          id: p.id || index,
          patientName: p.fullName || 'Unknown Name',
          fullName: p.fullName || 'Unknown Name',
          email: p.email || '',
          patientId: p.govId || 'N/A',
          date: p.dateBirth || new Date(Date.now() - Math.random() * 365 * 50 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
          sex: p.sex || 'N/A',
          age: p.dateBirth ? this.calculateAge(p.dateBirth) : 'N/A',
          diseases: p.diseases || 'N/A',
          status: p.status || 'Unknown',
          doctorName: p.doctorName || 'N/A'
        }))
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los pacientes'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    resetPacienteForm () {
      this.paciente = {
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
        international: false,
        photo: ''
      }
      this.photoFile = null
      this.isValidImage = false
      this.formErrors = {
        demographics: false,
        address: false,
        photo: false
      }
      this.photoRequired = false
    },
    nextTab () {
      if (this.activeTab === 0) {
        if (!this.validateDemographics()) {
          return
        }
      } else if (this.activeTab === 1) {
        if (!this.validateAddress()) {
          return
        }
      }
      this.activeTab++
    },
    validateDemographics () {
      const { fullName, email, mobile, dateBirth, motherTongue, govId, sex, maritalStatus } = this.paciente
      if (!fullName || !email || !mobile || !dateBirth || !motherTongue || !govId || !sex || !maritalStatus) {
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Please fill in all demographics fields',
          type: 'error'
        })
        this.formErrors.demographics = true
        return false
      }
      this.formErrors.demographics = false
      return true
    },
    validateAddress () {
      const { address, city, zipCode, state, country, district } = this.paciente
      if (!address || !city || !zipCode || !state || !country || !district) {
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Please fill in all address fields',
          type: 'error'
        })
        this.formErrors.address = true
        return false
      }
      this.formErrors.address = false
      return true
    },
    validatePhoto () {
      if (!this.paciente.photo) {
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Please upload a photo',
          type: 'error'
        })
        this.photoRequired = true
        this.formErrors.photo = true
        return false
      }
      this.photoRequired = false
      this.formErrors.photo = false
      return true
    },
    validateAndSave () {
      // Validate all sections
      const demographicsValid = this.validateDemographics()
      const addressValid = this.validateAddress()
      const photoValid = this.validatePhoto()

      if (!demographicsValid) {
        this.activeTab = 0
        return
      }

      if (!addressValid) {
        this.activeTab = 1
        return
      }

      if (!photoValid) {
        this.activeTab = 2
        return
      }

      // If all validations pass, proceed with saving
      this.savePatient()
    },
    async savePatient () {
      try {
        await this.$axios.post('/pacientes/create', this.paciente)
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Paciente Guardado Satisfactoriamente',
          type: 'success'
        })
        this.dialogAddPacient = false
        this.resetPacienteForm()
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
      this.resetPacienteForm()
      this.paciente = { ...this.paciente, ...patient }
      this.dialogEditPacient = true
    },
    async updatePatient () {
      try {
        await this.$axios.put(`/pacientes/update/${this.paciente.id}`, this.paciente)
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Paciente Actualizado Satisfactoriamente',
          type: 'success'
        })
        this.dialogEditPacient = false
        this.loadPacientes()
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el paciente'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    confirmDeletePatient () {
      this.patientToDelete = this.paciente
      this.confirmDeleteDialog = true
    },
    async executeDelete () {
      if (!this.patientToDelete || !this.patientToDelete.id) { return }

      try {
        await this.$axios.delete(`/pacientes/delete/${this.patientToDelete.id}`)
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Paciente Eliminado Con Éxito',
          type: 'success'
        })
        this.confirmDeleteDialog = false
        this.dialogEditPacient = false
        this.loadPacientes()
      } catch (error) {
        const errorMessage = error.message || 'Error al eliminar el paciente'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    deletePatient (patient) {
      this.patientToDelete = patient
      this.confirmDeleteDialog = true
    },
    formatDate (dateString) {
      if (!dateString) { return 'N/A' }
      try {
        const options = { year: '2-digit', month: 'short', day: 'numeric' }
        return new Date(dateString).toLocaleDateString('en-US', options)
      } catch (e) {
        return dateString
      }
    },
    calculateAge (dateString) {
      if (!dateString) { return 'N/A' }
      try {
        const birthDate = new Date(dateString)
        const today = new Date()
        let age = today.getFullYear() - birthDate.getFullYear()
        const m = today.getMonth() - birthDate.getMonth()
        if (m < 0 || (m === 0 && today.getDate() < birthDate.getDate())) {
          age--
        }
        return age
      } catch (e) {
        return 'N/A'
      }
    },
    getStatusColor (status) {
      const lowerStatus = status?.toLowerCase()
      if (lowerStatus === 'compilate' || lowerStatus === 'complete') {
        return 'green lighten-5'
      }
      if (lowerStatus === 'in-treatment') {
        return 'orange lighten-5'
      }
      return 'grey lighten-3'
    },
    getStatusChipTextColor (status) {
      const lowerStatus = status?.toLowerCase()
      if (lowerStatus === 'compilate' || lowerStatus === 'complete') {
        return 'green--text text--darken-3'
      }
      if (lowerStatus === 'in-treatment') {
        return 'orange--text text--darken-3'
      }
      return 'grey--text text--darken-1'
    },
    getStatusChipClasses (status) {
      const lowerStatus = status?.toLowerCase()
      if (lowerStatus === 'compilate' || lowerStatus === 'complete') {
        return 'status-chip-compilate'
      }
      if (lowerStatus === 'in-treatment') {
        return 'status-chip-in-treatment'
      }
      return 'status-chip-default'
    },
    getSimulatedPacientes () {
      return [
        { id: '018SyHTCaSHpNJnVQJV5', fullName: 'lolo', email: 's', mobile: 's', dateBirth: '2025-04-08', motherTongue: 's', govId: 's', sex: 's', maritalStatus: 's', address: 'ss', city: 's', zipCode: 's', state: 's', country: 's', district: 's', international: 'false', photo: null, diseases: 'Blood pressure', status: 'Compilate', doctorName: 'Dr. Smith' },
        { id: 2, fullName: 'SALVADOR ORTIZ VARGAS', email: 's.orti', dateBirth: '2024-05-06', govId: 's', sex: 's', diseases: 'Blood pressure', status: 'Compilate', doctorName: 'Dr. Jones', photo: null },
        { id: 3, fullName: 'ccc', email: 'ccc', dateBirth: '2017-01-02', govId: 'cc', sex: 'cc', diseases: 'Diabetes', status: 'In-Treatment', doctorName: 'Dr. Roy', photo: null },
        { id: 4, fullName: 'Phoenix Baker', email: 'phoenix', dateBirth: '1987-12-09', govId: 'PB123', sex: 'Male', diseases: 'Blood pressure', status: 'Compilate', doctorName: 'Dr. Imran Ali', photo: null },
        { id: 5, fullName: 'Lana Steiner', email: 'lana', dateBirth: '1984-12-23', govId: 'LS456', sex: 'Male', diseases: 'Diabetes', status: 'In-Treatment', doctorName: 'Dr. Mustag', photo: null }
      ]
    },
    openAddPatientDialog () {
      this.resetPacienteForm()
      this.dialogAddPacient = true
    }
  }
}
</script>

<style scoped>
.title {
  font-size: 24px;
  font-weight: 500;
  color: #101828;
}

.search-field {
  max-width: 300px;
}

.btnFilter {
  background-color: white !important;
  color: #344054 !important;
  margin-right: 10px !important;
  border: 1px solid #d0d5dd !important;
  border-radius: 8px !important;
  padding: 8px 14px !important;
  font-size: 14px !important;
  font-weight: 500 !important;
  text-transform: none !important;
  box-shadow: 0 1px 2px rgba(16, 24, 40, 0.05);
}

.btnFilter .v-icon {
  color: #667085;
}

.btnPatient {
  text-transform: none !important;
  color: white !important;
  border-radius: 8px !important;
  padding: 8px 14px !important;
  background-color: #7f56d9 !important;
  font-size: 14px !important;
  font-weight: 500 !important;
  box-shadow: 0 1px 2px rgba(16, 24, 40, 0.05);
}

.btnPatient .v-icon {
  margin-right: 4px;
}

.patients-table {
  border: 1px solid #eaecf0;
  border-radius: 8px;
  overflow: hidden;
}

.primary-header {
  background-color: #f8f9fc;
  border-bottom: 1px solid #eaecf0;
  padding: 16px 24px;
}

.date-display {
  white-space: nowrap;
  min-width: 100px;
  display: inline-block;
}

::v-deep .patients-table .v-data-table-header th {
  background-color: #f9fafb;
  color: #667085;
  font-size: 12px;
  font-weight: 500;
  text-transform: uppercase;
  padding: 12px 24px !important;
  border-bottom: 1px solid #eaecf0;
}

::v-deep .patients-table .v-data-table-header th .v-icon {
  color: #667085 !important;
  font-size: 16px;
}

::v-deep .patients-table tbody tr td {
  color: #667085;
  font-size: 14px;
  padding: 16px 24px !important;
  border-bottom: 1px solid #eaecf0;
  vertical-align: middle;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 0;
}

::v-deep .patients-table th:last-child,
::v-deep .patients-table td:last-child {
  min-width: 100px;
  width: 100px;
}

::v-deep .patients-table th:nth-child(1),
::v-deep .patients-table td:nth-child(1) {
  width: 48px;
  min-width: 48px;
}

::v-deep .patients-table th:nth-child(3),
::v-deep .patients-table td:nth-child(3) {
  min-width: 110px;
}

::v-deep .patients-table th:nth-child(4),
::v-deep .patients-table td:nth-child(4) {
  min-width: 80px;
}

::v-deep .patients-table th:nth-child(5),
::v-deep .patients-table td:nth-child(5) {
  min-width: 80px;
}

::v-deep .patients-table th:nth-child(6),
::v-deep .patients-table td:nth-child(6) {
  min-width: 120px;
}

::v-deep .patients-table th:nth-child(7),
::v-deep .patients-table td:nth-child(7) {
  min-width: 120px;
}

::v-deep .patients-table th:nth-child(8),
::v-deep .patients-table td:nth-child(8) {
  min-width: 140px;
}

::v-deep .patients-table {
  table-layout: fixed;
  width: 100%;
}

::v-deep .patients-table tbody tr:last-child td {
  border-bottom: none;
}

::v-deep .patients-table tbody tr td .font-weight-medium {
  color: #101828;
  font-weight: 500;
}

::v-deep .patients-table tbody tr td .text-caption {
  color: #667085;
}

::v-deep .patients-table .v-chip {
  font-weight: 500;
  padding: 2px 8px !important;
  height: 22px !important;
  font-size: 12px;
  border-radius: 16px !important;
}

::v-deep .patients-table .v-chip.status-chip-compilate {
  background-color: #ecfdf3 !important;
}

::v-deep .patients-table .v-chip.status-chip-in-treatment {
  background-color: #fffaeb !important;
}

::v-deep .patients-table .v-chip.status-chip-default {
  background-color: #f2f4f7 !important;
}

::v-deep .patients-table .v-chip .v-icon {
  font-size: 8px !important;
  line-height: 1;
  margin-right: 6px !important;
}

::v-deep .v-data-footer {
  border-top: 1px solid #eaecf0;
}

.pa-4 .v-btn {
  font-weight: 500;
  font-size: 14px;
  color: #667085;
  text-transform: none;
}

.pa-4 .v-btn--text {
  padding: 6px 8px;
}

.pa-4 .v-btn .v-icon {
  font-size: 18px;
}

.pa-4 .mx-1 {
  min-width: 36px;
  padding: 6px 4px !important;
}

.pa-4 .active-page {
  background-color: #f9f5ff;
  color: #6941c6 !important;
  border-radius: 6px;
}

::v-deep .v-data-table__selected {
  background: #f9fafb !important;
}

::v-deep .v-simple-checkbox .v-icon {
  color: #d0d5dd !important;
}

::v-deep .v-simple-checkbox .mdi-checkbox-marked {
  color: #7f56d9 !important;
}

.text-truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.action-buttons .v-btn {
  text-transform: none !important;
  font-weight: 500;
}

.form-label {
  font-size: 14px;
  font-weight: 500;
  color: #344054;
  margin-bottom: 6px;
}

.tab-container {
  border-bottom: 1px solid #e0e0e0;
  padding: 0;
  background-color: #f8f9fc;
}

::v-deep .v-tabs {
  padding: 0 16px;
  max-width: 800px;
  margin: 0 auto;
}

::v-deep .v-tab.tab-button {
  border-radius: 0 !important;
  margin: 0 3px;
  font-weight: 500;
  letter-spacing: 0.3px;
  color: #667085;
  height: 56px;
  min-width: 120px;
}

::v-deep .v-tab.tab-button:first-child {
  margin-left: 0;
}

::v-deep .v-tab.tab-button:last-child {
  margin-right: 0;
}

.active-tab {
  background-color: #0b1f61 !important;
  color: white !important;
  font-weight: 600 !important;
  border-radius: 4px 4px 0 0 !important;
}

.tab-content-container {
  min-height: 450px;
  padding: 24px !important;
  background-color: #ffffff;
}

.form-field-container {
  width: 100%;
}

.row-container {
  display: flex;
  width: 100%;
}

.row-container > .form-field-container {
  flex: 1;
}

.radio-group-container {
  background-color: #f3f4f8;
  padding: 8px 16px;
  border-radius: 8px;
}

.form-field {
  width: 100%;
}

.checkbox-container {
  padding: 8px 0;
}

.action-footer {
  border-top: 1px solid #e0e0e0;
}

.action-button, .next-button {
  background-color: #0e2579 !important;
  color: white !important;
  padding: 8px 24px !important;
  border-radius: 8px !important;
  font-weight: 500 !important;
  text-transform: none !important;
  box-shadow: 0 1px 2px rgba(16, 24, 40, 0.05) !important;
}

.back-button, .cancel-button {
  color: #344054 !important;
  font-weight: 500 !important;
  text-transform: none !important;
}
</style>
