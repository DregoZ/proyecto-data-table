<template>
  
    <!-- mx auto centra automaticamente y asigna un ancho según contenido -->
    <v-card class="mx-auto" min-width="80%" outlined style="padding:25px">
      
      <v-card outlined>
        <v-expansion-panels focusable>
          <v-expansion-panel>
            <v-expansion-panel-header>Filters</v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-row>
                <v-col cols="12" sm="6" md="4">
                  <v-menu
                    v-model="menu"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    min-width="290px"
                  >
                    <template v-slot:activator="{ on }">
                      <v-text-field
                        v-model="dateSearch"
                        label="Created between..."
                        prepend-icon="event"
                        readonly
                        v-on="on"
                      ></v-text-field>
                    </template>
                    <v-date-picker no-title v-model="dateSearch" range>
                      <v-spacer></v-spacer>
                      <v-btn text color="primary" @click="cancelDateSearch">Cancel</v-btn>
                      <v-btn text color="primary" @click="dateRangeSearch">OK</v-btn>
                    </v-date-picker>
                  </v-menu>
                </v-col>

                <v-col cols="12" sm="6" md="4">
                  <v-select
                    v-model="state"
                    :items="dropdown_state"
                    menu-props="auto"
                    hide-details
                    prepend-icon="mdi-set-none"
                    multiple
                    label="Filter by State"
                  ></v-select>
                </v-col>

                <v-col cols="12" sm="6" md="4">
                  <v-select
                    v-model="payment"
                    :items="dropdown_payment"
                    menu-props="auto"
                    hide-details
                    prepend-icon="mdi-currency-eur"
                    multiple
                    label="Filter by Payment"
                  ></v-select>
                </v-col>
              </v-row>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>

        <!-- data-table -->
        <v-data-table
          :headers="cabecera"
          :items="participantes"
          :sort-by="['firstname']"
          :sort-desc="[false, true]"
          :search="search"
          multi-sort
        >
          <!-- encabezado tabla: descripción, búsqueda, añadir... -->
          <template v-slot:top>
            <v-card-title>
              Members
              <v-spacer></v-spacer>

              <!-- botón añadir nuevo -->
              <v-flex class="mt-4 mb-3">
                <v-dialog v-model="dialog" max-width="750px">
                  <template v-slot:activator="{ on }">
                    <v-btn color="primary" dark class="mb-2" v-on="on">Add new member</v-btn>
                  </template>

                  <!-- popup New Data/Edit data-->
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
                            <!-- menu para date pick -->
                            <v-menu
                              ref="menu"
                              v-model="menu"
                              :close-on-content-click="false"
                              :return-value.sync="date"
                              transition="scale-transition"
                              offset-y
                              min-width="290px"
                            >
                              <template v-slot:activator="{ on }">
                                <v-text-field
                                  v-model="participanteEditar.created"
                                  label="Created in..."
                                  prepend-icon="event"
                                  readonly
                                  v-on="on"
                                ></v-text-field>
                              </template>
                              <v-date-picker
                                v-model="participanteEditar.created"
                                no-title
                                scrollable
                              >
                                <v-spacer></v-spacer>
                                <v-btn text color="primary" @click="menu = false">Cancel</v-btn>
                                <v-btn text color="primary" @click="$refs.menu.save(date)">OK</v-btn>
                              </v-date-picker>
                            </v-menu>
                          </v-col>

                          <!-- select para elegir tipo de pago -->
                          <v-col cols="12" sm="6" md="4">
                            <v-select
                              v-model="participanteEditar.paymentopt"
                              :items="dropdown_payment"
                              menu-props="auto"
                              label="Payment option"
                              hide-details
                              prepend-icon="mdi-currency-eur"
                              single-line
                            ></v-select>
                          </v-col>

                          <!-- select para state -->
                          <v-col cols="12" sm="6" md="4">
                            <v-select
                              v-model="participanteEditar.state"
                              :items="dropdown_state"
                              menu-props="auto"
                              label="State"
                              hide-details
                              prepend-icon="mdi-set-none"
                              single-line
                            ></v-select>
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

              <!-- search bar -->
              <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Search by email or id"
                single-line
                hide-details
              ></v-text-field>
            </v-card-title>
          </template>

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
          <!-- test CUSTOM FILTER -->
        </v-data-table>
      </v-card>
     
    </v-card>
  
</template>


<script>
export default {
  data: () => ({
    dialog: false,
    dropdown_payment: ["Cash", "Credit Card", "Sofort"],
    dropdown_state: ["Waiting List", "Completed", "Enrolled"],
    search: "",
    state: [],
    payment: [],
    dateSearch: [],
    date: new Date().toISOString().substr(0, 10),
    menu: false,
    modal: false,
    menu2: false,

    participantes: [
      {
        firstname: "Ken",
        lastname: "Masters",
        email: "srken@capcom.com",
        id: "0184",
        created: "2015-06-01",
        paymentopt: "Credit Card",
        state: "Waiting List"
      },
      {
        firstname: "Booker",
        lastname: "DeWitt",
        email: "booker@columbia.us",
        id: "1298",
        created: "2011-12-31",
        paymentopt: "Credit Card",
        state: "Completed"
      },
      {
        firstname: "Lara",
        lastname: "Croft",
        email: "mscroft@muchpounds.uk",
        id: "6547",
        created: "2004-02-25",
        paymentopt: "Sofort",
        state: "Enrolled"
      },
      {
        firstname: "Cammy",
        lastname: "White",
        email: "notadoll@shadaloo.th",
        id: "1248",
        created: "2018-07-12",
        paymentopt: "Credit Card",
        state: "Completed"
      },
      {
        firstname: "Jill",
        lastname: "Valentine",
        email: "jillsandwich@stars.rc",
        id: "3469",
        created: "1998-11-15",
        paymentopt: "Credit Card",
        state: "Enrolled"
      },
      {
        firstname: "Cloud",
        lastname: "Strife",
        email: "traumaboy@avalanche.md",
        id: "0345",
        created: "2009-10-09",
        paymentopt: "Cash",
        state: "Waiting List"
      },
      {
        firstname: "Nina",
        lastname: "Williams",
        email: "fatalblonde@hitmen.io",
        id: "5496",
        reated: "2015-07-20",
        paymentopt: "Sofort",
        state: "Waiting List"
      },
      {
        firstname: "Kazuya",
        lastname: "Mishima",
        email: "diedaddydie@gcorp.jp",
        id: "0354",
        created: "2016-01-21",
        paymentopt: "Credit Card",
        state: "Completed"
      },
      {
        firstname: "Nathan",
        lastname: "Drake",
        email: "darealdrake@search.com",
        id: "7841",
        created: "2014-04-12",
        paymentopt: "Cash",
        state: "Enrolled"
      },
      {
        firstname: "Samus",
        lastname: "Aran",
        email: "cutiepie@space.es",
        id: "9255",
        created: "2009-07-26",
        paymentopt: "Sofort",
        state: "Enrolled"
      },

      {
        firstname: "Mario",
        lastname: "Mario",
        email: "toomanyshrooms@donutplain.bw",
        id: "1003",
        created: "2011-11-11",
        paymentopt: "Cash",
        state: "Completed"
      },

      {
        firstname: "Mario",
        lastname: "Mario",
        email: "toomanyshrooms@donutplain.bw",
        id: "2015",
        created: "2011-11-11",
        paymentopt: "Cash",
        state: "Completed"
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
    cabecera() {
      return [
        {
          text: "First Name",
          value: "firstname",
          filterable: false
        } /*prop filterable para que no se encuentre en el search*/,

        { text: "Last Name", value: "lastname", filterable: false },

        { text: "Email", value: "email", filterable: true },

        { text: "Id", value: "id", filterable: true },

        /* date picker filter */
        {
          text: "Created",
          value: "created",
          filter: value => {
            if (!this.dateSearch[0]) return true;
            return (
              Date(this.dateSearch[0]) <= Date(value) &&
              Date(this.dateSearch[1]) >= Date(value)
            );
          }
        },

        {
          text: "Payment",
          value: "paymentopt",
          filter: value => {
            if (!this.payment[0]) return true;
            return (
              value == this.payment[0] ||
              value == this.payment[1] ||
              value == this.payment[2]
            );
          }
        },

        /* state filter */
        {
          text: "State",
          value: "state",
          filter: value => {
            if (!this.state[0]) return true;
            return (
              value == this.state[0] ||
              value == this.state[1] ||
              value == this.state[2]
            );
          }
        },

        /* edit / delete */
        { text: "Actions", value: "actions", sortable: false }
      ];
    },

    tituloFormulario() {
      return this.indexEditar === -1 ? "New data" : "Edit data";
    }
  },

  methods: {
    cancelDateSearch() {
      this.menu = false;
      this.dateSearch = [];
    },

    dateRangeSearch() {
      this.menu = false;
    },

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
        this.participanteEditar = Object.assign({}, this.defaultPart);
        this.indexEditar = -1;
      }, 3000);
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
        case "Cash":
          return "mdi-cash";

        case "Credit Card":
          return "mdi-credit-card";

        case "Sofort":
          return "mdi-bank";

        default:
          break;
      }
    },

    getStateIcon(item) {
      switch (item) {
        case "Completed":
          return { color: "orange", icon: "mdi-check" };

        case "Enrolled":
          return { color: "green", icon: "mdi-check-all" };

        case "Waiting List":
          return { color: "grey", icon: "mdi-dots-horizontal" };

        default:
          break;
      }
    }
  }
};
</script>

