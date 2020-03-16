<template>
  <!-- mx auto cemtra automaticamente y asigna un ancho según contenido -->
  <v-card class="mx-auto" outlined>
    <!-- encabezado tabla: descripción, búsqueda, añadir... -->

    <v-card-title>
      Participantes
      <v-spacer></v-spacer>
      <v-text-field
        v-model="search"
        append-icon="mdi-magnify"
        label="Search"
        single-line
        hide-details
      ></v-text-field>
      <v-spacer></v-spacer>

      <!-- botón añadir nuevo -->
      <v-flex class="mt-4 mb-3">
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark class="mb-2" v-on="on">Add new</v-btn>
          </template>

          <!-- popup -->
          <v-card>
            <v-card-title>
              <span class="headline">{{ tituloFormulario }}</span>
            </v-card-title>
            <!-- zona texto/datos -->
            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="participanteEditar.firstname" label="First Name"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="participanteEditar.lastname" label="Last Name"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="participanteEditar.email" label="Email"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="participanteEditar.created" label="Created"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="participanteEditar.paymentopt" label="Payment"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="participanteEditar.state" label="State"></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-flex>
    </v-card-title>

    <!-- JSON v-for keko in participantes... ?? ------------------------------------------------------->
    <!-- prop show-select para seleccionar uno/varios elementos -->
    <v-data-table
      :headers="cabecera"
      :items="participantes"
      :sort-by="['firstname']"
      :sort-desc="[false, true]"
      :search="search"
      multi-sort
      show-select
    >
      <!-- email format -->
      <template v-slot:item.email="{ item }">
        <a :href="'mailto:' + item.email">{{item.email}}</a>
      </template>
      
      <!-- iconos payment -->
      <template v-slot:item.paymentopt="{ item }">
        <v-icon>{{ getPaymentIcon(item.paymentopt) }}</v-icon>
      </template>

      <!-- iconos state -->
      <template v-slot:item.state="{ item }">
        <v-icon :color="getStateIcon(item.state).color">{{ getStateIcon(item.state).icon }}</v-icon>
      </template>

      <!-- iconos edit/delete -->
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">mdi-pencil</v-icon>
        <v-icon small @click="deleteItem(item)">mdi-delete</v-icon>
      </template>
    </v-data-table>
  </v-card>
</template>


<script>
//import AddNew from "./AddNew";

export default {
  data: () => ({
    dialog: false,
    search: "",
    cabecera: [
      {
        text: "First Name",
        align: "start",
        value: "firstname",
        filterable: false
      } /*prop filterable para que no se encuentre en el search*/,

      { text: "Last Name", value: "lastname", filterable: false },

      { text: "Email", value: "email", filterable: true },

      /* date format---- pendiente */
      { text: "Created", value: "created", filterable: false },

      { text: "Payment", value: "paymentopt", filterable: false },

      { text: "State", value: "state", filterable: false },

      /* edit / delete */
      { text: "Actions", value: "actions", sortable: false }
    ],

    participantes: [
      {
        firstname: "Ken",
        lastname: "Masters",
        email: "srken@capcom.com",
        created: "11/11/11",
        paymentopt: "ccard",
        state: "waitinglist"
      },
      {
        firstname: "Booker",
        lastname: "DeWitt",
        email: "booker@columbia.us",
        created: "11/11/11",
        paymentopt: "ccard",
        state: "completed"
      },
      {
        firstname: "Lara",
        lastname: "Croft",
        email: "mscroft@muchpounds.uk",
        created: "11/11/11",
        paymentopt: "sofort",
        state: "enrolled"
      },
      {
        firstname: "Cammy",
        lastname: "White",
        email: "notadoll@shadaloo.th",
        created: "11/11/11",
        paymentopt: "ccard",
        state: "completed"
      },
      {
        firstname: "Jill",
        lastname: "Valentine",
        email: "jillsandwich@stars.rc",
        created: "11/11/11",
        paymentopt: "ccard",
        state: "enrolled"
      },
      {
        firstname: "Cloud",
        lastname: "Strife",
        email: "traumaboy@avalanche.md",
        created: "11/11/11",
        paymentopt: "cash",
        state: "waitinglist"
      },
      {
        firstname: "Nina",
        lastname: "Williams",
        email: "fatalblonde@hitmen.io",
        reated: "11/11/11",
        paymentopt: "sofort",
        state: "waitinglist"
      },
      {
        firstname: "Kazuya",
        lastname: "Mishima",
        email: "diedaddydie@gcorp,jp",
        created: "11/11/11",
        paymentopt: "ccard",
        state: "completed"
      },
      {
        firstname: "Nathan",
        lastname: "Drake",
        email: "darealdrake@search.com",
        created: "11/11/11",
        paymentopt: "cash",
        state: "enrolled"
      },
      {
        firstname: "Samus",
        lastname: "Aran",
        email: "cutiepie@space.es",
        created: "11/11/11",
        paymentopt: "sofort",
        state: "enrolled"
      },

      {
        firstname: "Mario",
        lastname: "Mario",
        email: "toomanyshrooms@donutplain.bw",
        created: "11/11/11",
        paymentopt: "cash",
        state: "completed"
      },

      {
        firstname: "Mario",
        lastname: "Mario",
        email: "toomanyshrooms@donutplain.bw",
        created: "11/11/11",
        paymentopt: "cash",
        state: "completed"
      }
    ],

    indexEditar: -1,

    participanteEditar: {
      firstname: "",
      lastname: "",
      email: "",
      created: "",
      paymentopt: "",
      state: ""
    },

    defaultPart: {
      firstname: "",
      lastname: "",
      email: "",
      created: "",
      paymentopt: "",
      state: ""
    }
  }),

  computed: {
    tituloFormulario() {
      return this.indexEditar === -1 ? "New data" : "Edit data";
    }
  },

  methods: {
    editItem(item) {
      this.indexEditar = this.participantes.indexOf(item);
      this.participanteEditar = Object.assign({}, item); ///////////////////////////////////////////////////////////////
      this.dialog = true;
    },

    /* Borra el elemento de índice index */
    deleteItem(item) {
      const index = this.participantes.indexOf(item);
      confirm("Seguro que desea borrar este participante?") &&
        this.participantes.splice(index, 1);
    },

    close() {
      this.dialog = false;
      setTimeout(() => {
        this.indexEditar = Object.assign({}, this.defaultPart);
        this.indexEditar = -1;
      }, 300);
    },

    save() {
      if (this.indexEditar > -1) {
        Object.assign(
          this.participantes[this.indexEditar],
          this.participanteEditar
        );
      } else {
        this.participantes.push(this.participanteEditar);
      }
      this.close();
    },

    getPaymentIcon(item) {
      switch (item) {
        case "cash":
          return "mdi-cash";

        case "ccard":
          return "mdi-credit-card";

        case "sofort":
          return "mdi-bank";

        default:
          break;
      }
    },

    getStateIcon(item) {
      switch (item) {
        case "completed":
          return { color: "orange", icon: "mdi-check" };

        case "enrolled":
          return { color: "green", icon: "mdi-check-all" };

        case "waitinglist":
          return { color: "grey", icon: "mdi-dots-horizontal" };

        default:
          break;
      }
    }
  }
};
</script>

