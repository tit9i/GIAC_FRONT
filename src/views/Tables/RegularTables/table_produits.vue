<template>
    <b-card no-body>
        <b-card-header class="border-0">
          <b-row>
            <b-col class="col-6">
              <h3 class="mb-0">Produits</h3>
            </b-col>
            <b-col class="col-6 text-right">
              <base-button icon @click="handleAdd" variant="primary" size="sm" type="primary">
                <span class="btn-inner--icon"><i class="ni ni-shop"></i></span>
                <span class="btn-inner--text">ajouter</span>
              </base-button>
            </b-col>
          </b-row>
        </b-card-header>
      <vue-element-loading :active="show" spinner="bar-fade-scale" color="#2dce94" />
        <el-table class="table-responsive table"
                  header-row-class-name="thead-light"
                  :data="produit_list">
          <vue-element-loading :active="show" spinner="bar-fade-scale" color="#2dce94" />

            <el-table-column label="Nom"
                             prop="nom"
                             min-width="130px">
              <template v-slot="{row}">
                  <span class="status">{{row.nom}}</span>
              </template>
            </el-table-column>

            <el-table-column label="Description"
                             prop="description"
                             min-width="250px">
              <template v-slot="{row}">
                  <span class="status">{{row.description}}</span>
              </template>
            </el-table-column>

            <el-table-column label="Action"
                             prop="completion"
                             min-width="100px">
              <template v-slot="{row}">
                <span class="text-sm">
                  <a href="#" class="mx-1"  data-bs-toggle="tooltip" data-bs-original-title="Preview product" @click="toproduitDetail(row.id)">
                    <i class="fas fa-eye text-secondary" aria-hidden="true"></i>
                  </a>
                  <a href="#" class="mx-1" data-bs-toggle="tooltip" data-bs-original-title="Edit product" @click="handleEdit(row.id)">
                    <i class="fas fa-user-edit text-secondary" aria-hidden="true"></i>
                  </a>
                  <a href="javascript:;" class="mx-1"  data-bs-toggle="tooltip" data-bs-original-title="Delete product">
                    <i class="fas fa-trash text-secondary" aria-hidden="true"></i>
                  </a>
                </span>
              </template>
            </el-table-column>
        </el-table>

        <b-card-footer class="py-4 d-flex justify-content-end">
            <base-pagination v-model="currentPage" :per-page="10" :total="50"></base-pagination>
        </b-card-footer>
      <!--  ================== ADD produit MODAL =================== -->
      <modal :show.sync="modal" title="Ajouter un produit">
        <vue-element-loading :active="modal_show" spinner="bar-fade-scale" color="#2dce94" />
        <form ref="produit_form">
          <b-form-group label="nom">
            <b-input-group prepend="@">
              <b-form-input v-model="produit.nom" placeholder="nom" required></b-form-input>
            </b-input-group>
          </b-form-group>

          <b-form-group label="Description">
            <b-input-group>
              <b-form-textarea v-model="produit.description" required></b-form-textarea>
            </b-input-group>
          </b-form-group>
        </form>
        <template slot="footer">
          <base-button v-if="add" type="primary" @click="addproduit">Ajouter</base-button>
          <base-button v-else type="primary" @click="editproduit(produit.id)">Modifier</base-button>
          <base-button type="link" class="ml-auto" @click="modal = false">Close</base-button>
        </template>
      </modal>
    </b-card>
</template>
<script>
  import Resource from "../../../api/resource";
  import {Message, Table, TableColumn} from 'element-ui';
  const produitResource = new Resource('produits');
  import VueElementLoading from "vue-element-loading";

  export default {
    name: 'produit-list',
    components: {
      [Table.name]: Table,
      [TableColumn.name]: TableColumn,
      VueElementLoading
    },
    data() {
      return {
        list: [],
        currentPage: 1,
        produit: {},
        show: false,
        produit_list: [],
        add: true,
        modal: false,
        modal_show: false,
      };
    },
    created() {
      this.getproduits();
    },
    methods: {
    getproduits(){
      this.show = true
      produitResource.list()
      .then((response) => {
        this.produit_list = response.data;
      })
      .catch((error) => {
        console.log(error);
      })
      .finally(()=>{
        this.show = false;
      });
      console.log('produit LIST', this.produit_list);
    },
    addproduit(bvModalEvent){
      bvModalEvent.preventDefault();
      const valid = this.$refs['produit_form'].checkValidity();
      console.log('VALID VALUE ', valid);
      if (valid){
        this.modal_show = true;
        console.log(this.produit);
        produitResource.store(this.produit)
          .then((response) => {
            Message({
              message: 'produit ajout?? avec succes',
              type: 'success',
              duration: 5 * 1000,
            })
          })
          .catch((error) => {
            console.log(error);
          })
          .finally(() => {
            this.modal_show = false;
            this.modal = false;
            this.getproduits();
            this.$nextTick(() => {
              this.$bvModal.hide('modal-1')
            })
            console.log('DONE');
          });
      } else {
        Message({
          message: 'Veuillez svp renseigner les champs',
          type: 'error',
          duration: 5 * 1000
        })
        return false;
      }
    },
    handleAdd(){
      this.modal = true;
      this.add = true;
    },
    async handleEdit(id){
      const { data } = await produitResource.get(id);
      this.produit = data;
      this.modal = true;
      this.add = false;
    },
    toproduitDetail(id){
      this.$router.push({ path: '/produits/' + id });
    },
    editproduit(id){
      console.log('ID produit', id);
      this.modal_show = true;
      produitResource.update(id, this.produit)
      .then((response) => {
        Message({
          message: response.message || 'Modification effectu?? avec succ??s',
          type: 'success',
          duration: 5 * 1000,
        });
      })
      .catch((error) => {
        console.log(error);
      })
      .finally(() => {
        this.getproduits();
        this.modal = false;
        this.modal_show = false;
      })
    },
    }
  }
</script>
<style lang="scss" scoped>
.text-secondary {
  color: #8392ab!important;
}
</style>
