<template>
    <div class="animated fadeIn">
        <div v-if="loading" class="center-loader">
          <half-circle-spinner
            :animation-duration="1000"
            :size="90"
            color="#ff1d5e"
          />
        </div>
        <div v-else class="col-sm-12 col-md-12 col-lg-12">
            <div class="card border">
                <div class="card-header">
                    <div>
                        <div>
                            <button v-on:click='switchAddingPracticum' class="btn btn-primary pull-right"><i class="fa fa-plus"></i>Tambah Baru</button>
                        </div>
                    </div>
                </div>
                <div class="card-body">
                    <div v-if="switcher.editablePracticum=='on'">
                        <div v-if="switcher.editableMode=='add'">
                        <editable-practicum act='add' v-on:changelist='updateListPracticum' v-on:closediv='switchEditablePracticum'></editable-practicum>
                        </div>
                        <div v-else-if="switcher.editableMode=='edit'">
                        <editable-practicum act='edit' v-bind:practicum='selectedPracticum' v-on:changelist='updateListPracticum' v-on:closediv='switchEditablePracticum' ></editable-practicum>
                        </div>
                        <div  v-else-if="switcher.editableMode=='delete'">
                        <editable-practicum act='delete' v-bind:practicum='selectedPracticum' v-on:changelist='updateListPracticum' v-on:closediv='switchEditablePracticum' ></editable-practicum>
                        </div>
                    </div>
                    <div class="card">
                        <div class="card-header">
                            <div><i class="fa fa-align-justify"></i> Praktikum</div>
                        </div>
                        <div class="card-body">
                            <table aria-busy="false" aria-colcount="4" aria-rowcount="-1" class="table b-table table-striped table-responsive-sm" id="__BVID__696_">
                                <!---->
                                <!---->
                                <thead class="">
                                    <tr>
                                        <th aria-colindex="1" class="">Mata Kuliah</th>
                                        <th aria-colindex="2" class="">Judul Praktikum</th>
                                        <th aria-colindex="3" class="">Koordinator</th>
                                        <th aria-colindex="4" class="">Sunting</th>
                                        <th aria-colindex="5" class="">Hapus</th>
                                    </tr>
                                </thead>
                                <!---->
                                <paginate
                                      name="practicum-paginate"
                                      :list="practicums"
                                      :per="10"
                                      tag="tbody"
                                    >
                                    <!---->
                                    <tr v-for="(practicum, index) in paginated('practicum-paginate')" aria-rowindex="" class="">
                                        <td aria-colindex="1" class="">{{ practicum.course.courseName }}</td>
                                        <td aria-colindex="2" class="">{{ practicum.name }}</td>
                                        <td aria-colindex="3" class="" v-if="practicum.coordinatorAssistance==null">Belum ada Koordinator Asisten</td>
                                        <td aria-colindex="3" class="" v-else-if="practicum.coordinatorAssistance!=null">{{ practicum.coordinatorAssistance.name }}</td>
                                        <td aria-colindex="4" class=""><button v-on:click="editPracticum(practicum)" class="btn"><i class="fa fa-edit"></i>Sunting</button></td>
                                        <td aria-colindex="5" class=""><button v-on:click="deletePracticum(practicum)" class="btn btn-danger"><i class="fa fa-trash"></i>Hapus</button></td>
                                    </tr>
                                    <!---->
                                    <!---->
                                </paginate>
                                <tfoot>
                                    <paginate-links class="" for="practicum-paginate" :limit="1" 
                                      :show-step-links="true"
                                      :classes="{
                                        'ul': 'pagination',
                                        '.next > a': 'page-link',
                                        '.prev > a': 'page-link'
                                    }"></paginate-links>
                                </tfoot>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
  .center-loader {
    margin-left: 40%;
    margin-top: 20%;
  }
</style>

<script>
import { getAllPracticum as getAllPracticumAPI } from '@/api/practicum'
import { HalfCircleSpinner } from 'epic-spinners'
import EditablePracticum from './EditablePracticum'

export default {
  name: 'list-practicum',
  components: {
    EditablePracticum,
    HalfCircleSpinner
  },
  async mounted () {
    this.getAllPracticum()
    setTimeout(() => {
      this.loading = false
    }, 300)
  },
  data () {
    return {
      loading: true,
      practicums: {},
      switcher: {
        editablePracticum: 'off',
        editableMode: 'add',
        indexButton: '-99'
      },
      selectedPracticum: {},
      paginate: ['practicum-paginate']
    }
  },
  methods: {
    deletePracticum: function (practicum) {
      this.switcher.editableMode = 'delete'
      this.selectedPracticum = Object.assign({}, practicum)
      this.switchEditablePracticum()
    },
    editPracticum: function (practicum) {
      this.switcher.editableMode = 'edit'
      this.selectedPracticum = Object.assign({}, practicum)
      this.switchEditablePracticum()
    },
    switchAddingPracticum: function () {
      this.switcher.editableMode = 'add'
      this.switchEditablePracticum()
    },
    switchEditablePracticum: function () {
      if (this.switcher.editablePracticum === 'off') {
        this.switcher.editablePracticum = 'on'
      } else {
        this.switcher.editablePracticum = 'off'
      }
    },
    getAllPracticum: function () {
      getAllPracticumAPI().then(response => { this.practicums = response.data.content })
    },
    updateListPracticum: function (args) {
      this.getAllPracticum()
    }
  }
}
</script>
