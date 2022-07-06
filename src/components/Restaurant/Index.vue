<template>
  <div class="grid">
    <div class="col-12">
      <div class="card">
        <h4>Réstaurants</h4>

        <Toolbar class="mb-4">
          <template v-slot:end>
            <div class="my-2">
              <Button
                label="Ajouter"
                icon="pi pi-plus"
                class="p-button-success mr-2"
                @click="changerec('test')"
              />
            </div>
            <!-- <Button
              type="button"
              label="Filtre"
              icon="pi pi-filter"
              @click="toggle"
              class="p-button-info"
            /> -->
            <OverlayPanel
              ref="op"
              appendTo="body"
              :showCloseIcon="true"
              style="width: 600px"
            >
              <div class="card">
                <h5>Types de pannes</h5>

                <div class="p-fluid formgrid grid">
                  <div
                    v-for="type in typesDePannes"
                    :key="type"
                    class="field col-12 md:col-6"
                  >
                    <div>
                      <Checkbox
                        id="checkOption1"
                        name="option"
                        :value="type.id"
                        v-model="pannesValue"
                      />
                      <label for="checkOption1">{{ type.type }}</label>
                    </div>
                  </div>
                </div>
                <Divider v-if="!this.isTechnicien" />
                <h5 v-if="!this.isTechnicien">Technicien</h5>
                <MultiSelect
                  v-if="!this.isTechnicien"
                  v-model="multiselectechnicienstValue"
                  :options="techniciensfilter"
                  optionLabel="name"
                  placeholder="Sélectionner technicien"
                  :filter="true"
                >
                  <template #value="slotProps">
                    <div
                      class="
                        inline-flex
                        align-items-center
                        py-1
                        px-2
                        bg-primary
                        text-primary
                        border-round
                        mr-2
                      "
                      v-for="option of slotProps.value"
                      :key="option.code"
                    >
                      <div>{{ option.full_name }}</div>
                    </div>
                    <template
                      v-if="!slotProps.value || slotProps.value.length === 0"
                    >
                      <div class="p-1">Sélectionner technicien</div>
                    </template>
                  </template>
                  <template #option="slotProps">
                    <div class="flex align-items-center">
                      <div>{{ slotProps.option.full_name }}</div>
                    </div>
                  </template>
                </MultiSelect>
                <Divider />
                <h5>status</h5>

                <div class="p-fluid formgrid grid">
                  <div
                    v-for="status in statusFilter"
                    :key="status"
                    class="field col-12 md:col-6"
                  >
                    <div>
                      <Checkbox
                        id="checkOption1"
                        name="option"
                        :value="status"
                        v-model="statusValue"
                      />
                      <label for="checkOption1">{{ status }}</label>
                    </div>
                  </div>
                </div>
                <Button
                  label="appliquer"
                  @click="filter"
                  class="mr-2 mb-2 p-button-info"
                ></Button>
                <Button
                  label="Initialiser"
                  class="mr-2 mb-2"
                  @click="initfilterP"
                ></Button>
              </div>
            </OverlayPanel>
          </template>
          <template v-slot:start>
            <Button label='Tout' :badge="restaurantAll.length"   class="p-button-help p-button-text mr-2 mb-2" @click="changeres('Tout')" />
            <Button label="En attente" :badge="restaurantAtt.length" class="p-button-help p-button-text mr-2 mb-2"  @click="changeres('En attente')"/>
            <Button label="Non approuvée" :badge="restaurantNa.length" class="p-button-help p-button-text mr-2 mb-2" @click="changeres('Non approuvée')" />
            <Button label="Aprouvée" :badge="restaurantA.length" class="p-button-help p-button-text mr-2 mb-2"  @click="changeres('Aprouvée')"/>
          </template>
        </Toolbar>
        <DataTable
          :value="restaurant"
          :paginator="true"
          class="p-datatable-gridlines"
          :rows="20"
          dataKey="id"
          :rowHover="true"
          v-model:filters="filters1"
          :loading="loading1"
          :filters="filters1"
          responsiveLayout="scroll"
          :globalFilterFields="[
            'nom_etablissement',
            'name',
            'nom',
            'email',
            'tele',
            'fonction',
            'created_at',
            'updated_at',
          ]"
        >
          <template #header>
            <div class="flex justify-content-between flex-column sm:flex-row">
              <span class="p-input-icon-left mb-2">
                <i class="pi pi-search" />
                <!-- <InputText
                  v-model="filters1['global'].value"
                  placeholder="Keyword Search"
                  style="width: 100%"
                /> -->
              </span>
            </div>
          </template>
          <template #empty> Aucun restaurant trouvé. </template>
          <template #loading>
            Chargement des données des restaurants. Veuillez patienter.
          </template>

          <Column
            field="name"
            header="Restaurant"
            style="min-width: 4rem"
            :sortable="true"
          >
            <template #body="{ data }">
              {{ data.nom_etablissement }}
            </template>
          </Column>
          <Column
            field="Catégorie"
            header="Catégorie"
            style="min-width: 4rem"
            :sortable="true"
          >
            <template #body="{ data }">
              {{ data.name }}
            </template>
          </Column>
            <Column
            header="Status"
            field="status"
            style="min-width: 12rem"
            :sortable="true"
          >
            <template #body="{data}">
                            <span v-if="data.valide == null" :class="'customer-badge status-proposal' ">En Attente</span>
                              <span v-if="data.valide == 0" :class="'customer-badge status-unqualified' ">Non approuvée</span>
                                <span v-if="data.valide == 1" :class="'customer-badge status-qualified' ">Approuvée</span>
                        </template>
          </Column>
          <Column
            field="nom"
            header="Interloccuteur"
            style="min-width: 12rem"
            :sortable="true"
          >
            <template #body="{ data }">
              {{ data.nom }}
            </template>
          </Column>
          <Column
            field="email"
            header="Email"
            style="min-width: 5rem"
            :sortable="true"
          >
            <template #body="{ data }">
              {{ data.email }}
            </template>
          </Column>
          <Column
            field="tele"
            header="Téléphone"
            style="min-width: 5rem"
            :sortable="true"
          >
            <template #body="{ data }">
              {{ data.tele }}
            </template>
          </Column>
          <Column
            field="fonction"
            header="Fonction"
            style="min-width: 10rem"
            :sortable="true"
          >
            <template #body="{ data }">
              {{ data.fonction }}
            </template>
          </Column>
          <Column
            field="full_name"
            header="Utilisateur"
            style="min-width: 12rem"
            :sortable="true"
          >
            <template #body="{ data }">
              {{ data.full_name }}
            </template>
          </Column>
          <Column
            field="created_at"
            header="Date de création"
            style="min-width: 12rem"
            :sortable="true"
          >
            <template #body="{ data }">
              {{ data.created_at }}
            </template>
          </Column>
           
        
          <Column header="Status" style="min-width: 10rem">
            <template #body="{ data }">
             <span class="p-buttonset">

             </span>
                <br>
              <span class="p-buttonset">
				<Button v-if="data.valide == null" label="Approuver" class="p-button-success mr-2 mb-2" v-on:click="approuverRestaurant(data.id)"/>
				<Button v-if="data.valide == null" label="Refuser" @click="showdialgadd(data.id)" class="p-button-danger mr-2 mb-2" />
              </span>
            </template>
          </Column>
             <Column header="Actions" style="min-width: 18rem">
            <template #body="{ data }">
             <span class="p-buttonset">

             </span>
                <br>
              <span class="p-buttonset">
                <Button
                  style="margin-right: 4px"
                  icon="pi pi-eye"
                  class="p-button-raised p-button-rounded p-button-success"
                  @click="showReclamation(data)"
                />
                <Button
                  
                  style="margin-right: 4px"
                  icon="pi pi-user-edit"
                  class="p-button-raised p-button-rounded p-button-info"
                  @click="updateReclamation(data.id)"
                />
                <Button
                  style="margin-right: 4px"
                  @click="deleteRestaurant(data.id)"
                  icon="pi pi-trash"
                  class="p-button-raised p-button-rounded p-button-danger"
                />
                
              </span>
            </template>
          </Column>
        </DataTable>
       
       <Dialog
          v-model:visible="dialogOfnote"
          :style="{ width: '500px' }"
          header="Refuser restaurant"
          :modal="true"
          class="p-fluid"
        >
          <div class="card">
            <div class="p-fluid formgrid grid">
              <div class="field col-12 md:col-12">
                <label for="city">Note </label>
                <InputText
                  v-model="note"
                  type="text"
                  :class="{ 'is-invalid': errorsAdd['note'] }"
                />

                <small class="p-error" v-if="errorsAdd['note']">
                  {{ errorsAdd["note"] }}
                </small>
              </div>
              
            </div>
          </div>
          <template #footer>
            <Button
              label="Approuver"
              icon="pi pi-times"
              class="p-button-text"
              @click="dialogOfnote = false"
            />
             <Button
              label="Modifier"
              icon="pi pi-check"
              class="p-button-text"
              @click="checkformup"
            />
          </template>
        </Dialog>
      </div>
    </div>
  </div>
 
</template>

<script>
import Axios from "axios";
import Swal from "sweetalert2";


export default {
  data() {
    return {
      idrestaurant:null,
      user: null,
      isTechnicien: false,
      clientDisabled: false,
      dateshow: true,
      errorsPlanifier: false,
      dialogOfPlanifier: false,
      recvalidatemsg: null,
      recvalidate: null,
      techvalidates: null,
      techvalidated: null,
      dialogOfvalid: false,
      statusFilter: ["terminée", "non terminée", "planifiée"],
      statusValue: null,
      switchValue: null,
      multiselectechnicienstValue: null,
      multiselectechnicienstValueid: [],
      techniciensfilter: [],
      typesDePannes: [],
      pannesValue: [],
      clientToShow: null,
      previewImages: [],
      previewImagesupdate1: null,
      previewImagesupdate2: null,
      previewImagesupdate3: null,
      images: [],
      Permissions: {
        update: false,
        delete: false,
        add: false,
        show: false,
        valider: false,
        planifier: false,
      },
      note:null,
      dialogOfnote:false,
      restaurantA: [],
      restaurantAtt: [],
      restaurantNa: [],
      restaurant: [],
      restaurantAll: [],
      reclamationsActif: true,
      techniciens: [],
      clients: [],
      reclamationsAdd: [],
      parcs: [],
      hours: [],
      hourstst: true,
      dialogOfAdd: false,
      client: null,
      parc: null,
      hour: null,
      TypePanne: null,
      disabledtest: true,
      reclamationAddObj: {
        client_id: null,
        typeRec_id: null,
        parc_id: null,
        message: null,
        messagePrv: null,
        file1: null,
        file2: null,
        file3: null,
        status: null,
        etat: null,
        ref: null,
        planiyed_at: null,
      },
      planiyed_at: null,
      minDateValue: null,
      invalidDates: [],
      filters1: null,
      loading1: true,
      display: false,
      errorimage: false,
      errorsAdd: [],
      errorsUp: [],
      dialogOfUpdate: false,
      hourupdate: false,
      deletedurl: [],
      dialogOfShow: false,
    };
  },
  mounted() {
    // if (localStorage.role_id == 2 || localStorage.role_id == 4) {
    //   this.isTechnicien = true;

    //   this.user = JSON.parse(localStorage.user);
    // }
    // this.Permissions.valider = localStorage.permissionNames.includes(
    //   "valider reclamation"
    // );
    // this.Permissions.update =
    //   localStorage.permissionNames.includes("update reclamation");
    // this.Permissions.delete =
    //   localStorage.permissionNames.includes("delete reclamation");
    // this.Permissions.add =
    //   localStorage.permissionNames.includes("create reclamation");
    // this.Permissions.planifier = localStorage.permissionNames.includes(
    //   "plannifier reclamation"
    // );
    // this.Permissions.show =
    //   localStorage.permissionNames.includes("show reclamation");

    // if (this.Permissions.show) {
    //   this.getReclamation();
    // } else {
    //   this.$router.push("/");
    // }
    this.getrestaurant();
  },

 
 
  methods: {
     showdialgadd(id) {
      this.idrestaurant = id;
      this.ititform();
      this.dialogOfnote = true;
    },
    ititform() {
      this.note = null;
    },
    checkformup(){
        let checked = true;
        this.errorsAdd = [];
        if (this.note == null || this.note == "") {
            this.errorsAdd["note"] = "Note est obligatoire.";
            checked = false;
        }
        console.log(checked);
        if (checked) {
            this.notapprouvRestaurant();
        }
    },
    changeres(value){
        if(value == "Tout"){
            this.restaurant = this.restaurantAll;
        }
        if(value == "En attente"){
            this.restaurant = this.restaurantAtt;
        }
        if(value == "Non approuvée"){
            this.restaurant = this.restaurantNa;
        }
        if(value == "Aprouvée"){
            this.restaurant = this.restaurantA;
        }
    },
    approuverRestaurant(id){
        Swal.fire({
                icon: "warning",
                title: "Vous êtes sûr de vouloir approuver ce restaurant ?",

                showCancelButton: true,
                confirmButtonText: "Approuver",
                cancelButtonText: "Annuler",
            }).then((result) => {
                /* Read more about isConfirmed, isDenied below */
                if (result.isConfirmed) {
                    Axios.post("/validationRestaurant/" + id,{"valide":1})
                    .then((rs) => {
                        console.log(rs.data);
                        this.getrestaurant();
                        Swal.fire({
                            icon: "success",
                            title: rs.data.message,
                            showConfirmButton: false,
                            timer: 1500,
                        });
                    });
                }
            });
    },
    notapprouvRestaurant(){
                    Axios.post("/validationRestaurant/" + this.idrestaurant,{"valide":0,"noteValidation":this.note})
                    .then((response) => {
                        this.dialogOfnote = false;
                        console.log("response", response);
                        Swal.fire({
                            icon: "success",
                            title: "Restaurant refusée avec succès",
                            showConfirmButton: false,
                            timer: 1500,
                        });
                        this.getrestaurant();
                        })
                        .catch((err) => {
                        if (err.response.data.message) {
                             Swal.fire("Erreur d'authentification", err.response.data.message);
                        } else {
                             Swal.fire("Erreur d'authentification", err.message);
                        }
        });
                
        
    },
     getrestaurant() {
            Axios.get("getRestaurantAdmin")
                .then((response) => {
                    if (response.data.status == "success") {
                       this.restaurant = response.data.restaurant;
                       this.restaurantA = response.data.restaurantValide;
                       this.restaurantAtt = response.data.restaurantEnAttente;
                       this.restaurantNa = response.data.restaurantNonValide;
                       this.restaurantAll=response.data.restaurant;

                        console.log(this.restaurant);
                        this.loading1 = false;
                    } else {
                        Swal.fire({
                            title: "Erreur",
                            text: response.data.message,
                            icon: "error",
                            confirmButtonClass: "btn btn-secondary",
                        });
                    }
                })
                .catch((error) => {
                    this.errorMessage = error.message;
                    console.error("There was an error!", error);
                });
        },
        deleteRestaurant(id) {
            Swal.fire({
                icon: "warning",
                title: "Vous êtes sûr de vouloir continuer ?",

                showCancelButton: true,
                confirmButtonText: "supprimer",
                cancelButtonText: "Annuler",
            }).then((result) => {
                /* Read more about isConfirmed, isDenied below */
                if (result.isConfirmed) {
                    Axios.post("/deleteRestaurant/" + id)
                    .then((rs) => {
                        console.log(rs.data);
                        this.getrestaurant();
                        Swal.fire({
                            icon: "success",
                            title: rs.data.message,
                            showConfirmButton: false,
                            timer: 1500,
                        });
                    });
                }
            });
        },
  },
};
</script>
<style scoped lang="scss">
@import "../../assets/demo/badges.scss";
@import "../../assets/demo/badges.scss";
::v-deep(.p-datatable-frozen-tbody) {
  font-weight: bold;
}

::v-deep(.p-datatable-scrollable .p-frozen-column) {
  font-weight: bold;
}
.is-invalid {
  border-color: #e24c4c;
}
.p-inputtext.p-invalid.p-component {
  border-color: #e24c4c;
}
.imagePreviewWrapper {
  width: 250px;
  height: 250px;
  display: block;
  cursor: pointer;
  margin: 0 auto 30px;
  background-size: cover;
  background-position: center center;
}
</style>
