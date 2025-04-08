<template>
  <v-container>
    <h1 class="text-4 mb-4">
      Gestión de Usuarios
    </h1>
    <!-- Renglón con el botón crear nuevo usuario -->
    <v-row align="center" justify="end">
      <v-btn color="primary" @click="openDialog('create')">
        <v-icon left>
          mdi-account-plus
        </v-icon>
        Crear Usuario
      </v-btn>
    </v-row>

    <!-- Tabla de los Usuarios -->
    <v-data-table
      :headers="headers"
      :items="usuarios"
      :items-per-page="10"
      dense
      class="mt-4"
    >
      <template #[`item.actions`]="{item}">
        <!-- Editar -->
        <v-icon small color="primary" title="Editar" @click="openDialog('edit', item)">
          mdi-pencil
        </v-icon>
        <!-- Eliminar -->
        <v-icon small color="red" title="Borrar" @click="openDialog('delete', item)">
          mdi-delete
        </v-icon>
      </template>
    </v-data-table>

    <!-- Dialog para crear o editar un usuario -->
    <v-dialog v-model="dialog" persistent max-width="700px">
      <v-card color="indigo lighten-5">
        <v-card-title>
          <span class="text-h6">
            {{ dialogMode === 'create' ? 'Crear Usuario': dialogMode === 'edit' ? 'Editar Usuario' : 'Eliminar Usuario' }}
          </span>
        </v-card-title>
        <v-card-text>
          <v-form v-if="dialogMode !== 'delete'" ref="form" v-model="valid">
            <v-row>
              <v-col cols="4">
                <v-text-field v-model="userData.nombre" label="Nombre" required />
              </v-col>
              <v-col cols="4">
                <v-text-field v-model="userData.apaterno" label="A. Paterno" required />
              </v-col>
              <v-col cols="4">
                <v-text-field v-model="userData.amaterno" label="A. Materno" required />
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="12">
                <v-text-field v-model="userData.direccion" label="Dirección" required />
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="4">
                <v-text-field v-model="userData.telefono" label="Teléfono" required />
              </v-col>
              <v-col cols="4">
                <v-select
                  v-model="userData.estado"
                  :items="estados.map(estado => estado.nombre)"
                  label="Selecciona un estado"
                  required
                  @change="updateCities"
                />
              </v-col>
              <v-col cols="4">
                <v-select
                  v-model="userData.ciudad"
                  :items="ciudades"
                  label="Selecciona una Ciudad"
                  :disabled="!userData.estado"
                  required
                />
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="4">
                <v-text-field v-model="userData.usuario" label="Usuario" required />
              </v-col>
              <v-col cols="4">
                <v-text-field v-model="userData.password" label="Password" type="password" required />
              </v-col>
              <v-col cols="4">
                <v-select v-model="userData.rol" :items="roles" required label="Rol" />
              </v-col>
            </v-row>
          </v-form>
          <div v-else>
            ¿Estás seguro de que deseas eliminar al usuario {{ selectedUser?.nombre }}?
          </div>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="closeDialog">
            Cancelar
          </v-btn>
          <v-btn :disabled="dialogMode !== 'delete' && !valid" @click="dialogMode === 'delete' ? deleteUser() : saveUser()">
            {{ dialogMode === 'delete' ? 'Eliminar' : 'Guardar' }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  layout: 'principal',
  middleware: 'auth',
  data () {
    return {
      usuarios: [],
      headers: [
        { text: 'Nombre', value: 'nombre' },
        { text: 'A. Paterno', value: 'apaterno' },
        { text: 'A. Materno', value: 'amaterno' },
        { text: 'Usuario', value: 'usuario' },
        { text: 'Rol', value: 'rol' },
        { text: 'Acciones', value: 'actions', sortable: false }
      ],
      roles: ['admin', 'contabilidad', 'recursos', 'secretaria'],
      valid: false,
      dialog: false,
      confirmDialog: false,
      dialogMode: 'create', // create - edit8
      userData: {
        nombre: '',
        apaterno: '',
        amaterno: '',
        direccion: '',
        telefono: '',
        ciudad: '',
        estado: '',
        usuario: '',
        password: '',
        rol: 'contabilidad'
      },
      estados: [
        {
          nombre: 'Aguascalientes',
          ciudades: ['Aguascalientes', 'Calvillo', 'Jesús María', 'Rincón de Romos', 'Pabellón de Arteaga']
        },
        {
          nombre: 'Baja California',
          ciudades: ['Mexicali', 'Tijuana', 'Ensenada', 'Rosarito', 'Tecate']
        },
        {
          nombre: 'Baja California Sur',
          ciudades: ['La Paz', 'Los Cabos', 'Ciudad Constitución', 'Loreto', 'Santa Rosalía']
        },
        {
          nombre: 'Campeche',
          ciudades: ['Campeche', 'Ciudad del Carmen', 'Champotón', 'Escárcega', 'Calkiní']
        },
        {
          nombre: 'Chiapas',
          ciudades: ['Tuxtla Gutiérrez', 'San Cristóbal de las Casas', 'Tapachula', 'Comitán', 'Chiapa de Corzo']
        },
        {
          nombre: 'Chihuahua',
          ciudades: ['Chihuahua', 'Ciudad Juárez', 'Cuauhtémoc', 'Delicias', 'Parral']
        },
        {
          nombre: 'Ciudad de México',
          ciudades: ['Álvaro Obregón', 'Coyoacán', 'Cuauhtémoc', 'Iztapalapa', 'Miguel Hidalgo']
        },
        {
          nombre: 'Coahuila',
          ciudades: ['Saltillo', 'Torreón', 'Monclova', 'Piedras Negras', 'Acuña']
        },
        {
          nombre: 'Colima',
          ciudades: ['Colima', 'Manzanillo', 'Tecomán', 'Villa de Álvarez', 'Comala']
        },
        {
          nombre: 'Durango',
          ciudades: ['Durango', 'Gómez Palacio', 'Lerdo', 'Ciudad Lerdo', 'El Salto']
        },
        {
          nombre: 'Guanajuato',
          ciudades: ['Guanajuato', 'León', 'Irapuato', 'Celaya', 'Salamanca']
        },
        {
          nombre: 'Guerrero',
          ciudades: ['Acapulco', 'Chilpancingo', 'Iguala', 'Taxco', 'Zihuatanejo']
        },
        {
          nombre: 'Hidalgo',
          ciudades: ['Pachuca', 'Tulancingo', 'Tizayuca', 'Huichapan', 'Apan']
        },
        {
          nombre: 'Jalisco',
          ciudades: ['Guadalajara', 'Zapopan', 'Tlaquepaque', 'Tonalá', 'Puerto Vallarta']
        },
        {
          nombre: 'Estado de México',
          ciudades: ['Toluca', 'Ecatepec', 'Nezahualcóyotl', 'Naucalpan', 'Tlalnepantla']
        },
        {
          nombre: 'Michoacán',
          ciudades: ['Morelia', 'Uruapan', 'Zamora', 'Lázaro Cárdenas', 'Apatzingán']
        },
        {
          nombre: 'Morelos',
          ciudades: ['Cuernavaca', 'Jiutepec', 'Temixco', 'Cuautla', 'Yautepec']
        },
        {
          nombre: 'Nayarit',
          ciudades: ['Tepic', 'Xalisco', 'Santiago Ixcuintla', 'Compostela', 'Bahía de Banderas']
        },
        {
          nombre: 'Nuevo León',
          ciudades: ['Monterrey', 'Guadalupe', 'San Nicolás de los Garza', 'Apodaca', 'Santa Catarina']
        },
        {
          nombre: 'Oaxaca',
          ciudades: ['Oaxaca de Juárez', 'Salina Cruz', 'Juchitán de Zaragoza', 'Huajuapan de León', 'Puerto Escondido']
        },
        {
          nombre: 'Puebla',
          ciudades: ['Puebla de Zaragoza', 'Tehuacán', 'San Martín Texmelucan', 'Atlixco', 'Cholula']
        },
        {
          nombre: 'Querétaro',
          ciudades: ['Querétaro', 'San Juan del Río', 'Corregidora', 'El Marqués', 'Tequisquiapan']
        },
        {
          nombre: 'Quintana Roo',
          ciudades: ['Cancún', 'Chetumal', 'Playa del Carmen', 'Cozumel', 'Tulum']
        },
        {
          nombre: 'San Luis Potosí',
          ciudades: ['San Luis Potosí', 'Soledad de Graciano Sánchez', 'Ciudad Valles', 'Matehuala', 'Rioverde']
        },
        {
          nombre: 'Sinaloa',
          ciudades: ['Culiacán', 'Mazatlán', 'Los Mochis', 'Guasave', 'Navolato']
        },
        {
          nombre: 'Sonora',
          ciudades: ['Hermosillo', 'Ciudad Obregón', 'Nogales', 'Guaymas', 'Navojoa']
        },
        {
          nombre: 'Tabasco',
          ciudades: ['Villahermosa', 'Cárdenas', 'Comalcalco', 'Huimanguillo', 'Macuspana']
        },
        {
          nombre: 'Tamaulipas',
          ciudades: ['Ciudad Victoria', 'Reynosa', 'Matamoros', 'Nuevo Laredo', 'Tampico']
        },
        {
          nombre: 'Tlaxcala',
          ciudades: ['Tlaxcala', 'Apizaco', 'Huamantla', 'Chiautempan', 'Calpulalpan']
        },
        {
          nombre: 'Veracruz',
          ciudades: ['Xalapa', 'Veracruz', 'Córdoba', 'Poza Rica', 'Orizaba']
        },
        {
          nombre: 'Yucatán',
          ciudades: ['Mérida', 'Valladolid', 'Tizimín', 'Progreso', 'Ticul']
        },
        {
          nombre: 'Zacatecas',
          ciudades: ['Zacatecas', 'Fresnillo', 'Guadalupe', 'Jerez', 'Calera']
        }
      ],
      ciudades: [],
      selectedUser: null
    }
  },
  mounted () {
    this.fetchUsers()
  },
  methods: {
    async fetchUsers () {
      try {
        const response = await this.$axios.get('/users')
        // console.log('@@@ response => ', response)
        this.usuarios = response.data.users
      } catch (error) {
        const errorMessage = error.message || 'Error al cargar los usuarios'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    openDialog (mode, user = null) {
      this.dialogMode = mode
      if (mode === 'edit' && user) {
        this.dialog = true
        user.password = ''
        this.userData = { ...user }
      } else if (mode === 'create') {
        this.dialog = true
        this.userData = {
          nombre: '',
          apaterno: '',
          amaterno: '',
          direccion: '',
          telefono: '',
          ciudad: '',
          estado: '',
          usuario: '',
          password: '',
          rol: 'contabilidad'
        }
      } else {
        this.dialog = true
        this.selectedUser = user
      }
    },
    closeDialog () {
      this.dialog = false
    },
    saveUser () {
      if (this.dialogMode === 'create') {
        this.createUser()
      } else {
        this.updateUser()
      }
    },
    async createUser () {
      try {
        await this.$axios.post('/users/create', this.userData)
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Creado Con Éxito',
          type: 'success'
        })
        this.fetchUsers()
        this.closeDialog()
      } catch (error) {
        const errorMessage = error.message || 'Error al crear el usuario'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async updateUser () {
      try {
        await this.$axios.put(`/users/update/${this.userData.id}`, this.userData)
        this.$store.dispatch('alert/triggerAlert', {
          message: 'Usuario Actualizado Con Éxito',
          type: 'success'
        })
        this.fetchUsers()
        this.closeDialog()
      } catch (error) {
        const errorMessage = error.message || 'Error al actualizar el usuario'
        this.$store.dispatch('alert/triggerAlert', {
          message: errorMessage,
          type: 'error'
        })
      }
    },
    async deleteUser () {
      if (this.selectedUser) {
        try {
          await this.$axios.delete(`/users/delete/${this.selectedUser.id}`)
          this.$store.dispatch('alert/triggerAlert', {
            message: 'Usuario Eliminado Con Éxito',
            type: 'success'
          })
          this.fetchUsers()
          this.closeDialog()
        } catch (error) {
          const errorMessage = error.message || 'Error al eliminar el usuario'
          this.$store.dispatch('alert/triggerAlert', {
            message: errorMessage,
            type: 'error'
          })
        }
      }
      this.confirmDialog = false
    },
    updateCities () {
      const estadoSeleccionado = this.estados.find(estado => estado.nombre === this.userData.estado)
      this.ciudades = estadoSeleccionado ? estadoSeleccionado.ciudades : []
      this.userData.ciudad = ''
    }
  }
}
</script>

<style scoped>

</style>
