<template>
    <div class="fadeIn">
      <b-modal title="Modal title" hide-footer size="lg" v-model="modal" 
      hide-header no-close-on-backdrop no-close-on-esc>
        <div class="card card-primary">
            <div class="card-header">
                <div v-if="mode=='add'"><strong>Tambah Tugas Baru</strong>
                </div>
                <div v-else-if="mode=='edit'"><strong>Edit Tugas - {{ task.title }}</strong>
                </div>
                <div v-else-if="mode=='delete'"><strong>Hapus Tugas - {{ task.title }}</strong>
                </div>
            </div>
            <div class="card-body">
                <div v-if="mode=='delete'"><span> Apakah anda yakin ingin menghapus tugas ini? </span></div>
                <div role="group" class="b-form-group form-group">
                    <div class="form-row"><label for="horizPass" class="col-sm-3 col-form-label">Judul Tugas</label>
                        <div class="col-sm-9">
                        <input v-if="mode=='delete'" v-model="task.title" type="text" placeholder="Judul tugas.." disabled class="form-control">
                        <input v-else v-model="task.title" type="text" placeholder="Judul tugas.." class="form-control">
                        <!----><small class="form-text text-muted">Masukkan Judul Tugas</small></div>
                    </div>
                </div>
                <div role="group" class="b-form-group form-group">
                    <div class="form-row"><label for="horizPass" class="col-sm-3 col-form-label">Deskripsi Tugas</label>
                        <div class="col-sm-9">
                        <div v-if="mode=='delete'"><p v-html="task.description"></p></div>
                        <div v-else><wysiwyg v-model="task.description" /></div>
                        <!----><small class="form-text text-muted">Deskripsikan Detail Tugas Serinci Mungkin</small></div>
                    </div>
                </div>
                <div role="group" class="b-form-group form-group">
                    <div class="form-row"><label for="horizPass" class="col-sm-3 col-form-label">Waktu Pengerjaan</label>
                        <div class="col-sm-9">
                        <div v-if="mode=='delete'"> <p>Mulai : {{ task.createdDate }} - Selesai : {{ task.dueDate }}</p> </div>
                        <date-picker v-else v-on:confirm="assignmentDatePicker()" v-model="temp.selectedDate" lang="en" range type="datetime" 
                        format="yyyy-MM-dd" confirm></date-picker>
                        <!----><small class="form-text text-muted">Batas Mulai Sampai Selesai</small></div>
                    </div>
                </div>
                <div role="group" class="b-form-group form-group">
                    <div class="form-row"><label for="horizPass" class="col-sm-3 col-form-label">Keterlambatan</label>
                        <div class="col-sm-9">
                        <div v-if="mode=='delete'">
                            <p v-if="task.allowLate">Ya</p>
                            <p v-else>Tidak</p>
                        </div>
                        <div v-else>
                            <label class="switch switch-text switch-primary">
                                <input type="checkbox" class="switch-input" checked v-model="task.allowLate">
                                <span class="switch-label" data-on="Ya" data-off="Tdk"></span>
                                <span class="switch-handle"></span>
                            </label>
                        </div>
                        <!----><small class="form-text text-muted">Perbolehkan Pengumpulan Seusai Keterlambatan Peserta</small></div>
                    </div>
                </div>
                <div role="group" class="b-form-group form-group">
                    <div class="form-row"><label for="horizPass" class="col-sm-3 col-form-label">Laman Pengerjaan</label>
                        <div class="col-sm-3" v-if="mode!='delete'">
                            <input type="text" placeholder="Judul lampiran.." class="form-control" v-model="temp.assignment.description">
                        </div>
                        <div class="col-sm-3" v-if="mode!='delete'">
                            <select name="" id="" class="form-control" v-model="temp.assignment.fileAllowed">
                                <option value="document">Dokumen (.doc/.docx/.pdf)</option>
                                <option value="sourcecode">Kode Program (.java/.php/.py)</option>
                                <option value="text">Input Manual (Form Pengisian Tambahan)</option>
                            </select>
                        </div>
                        <div class="col-sm-2" v-if="mode!='delete'">
                            <button v-on:click="addTemporaryAssignment()" class="btn btn-primary"><i class="fa fa-plus"></i></button>
                        </div>
                    </div>
                </div>
                <div role="group" class="b-form-group form-group" v-if="task.assignments.length>0" v-for="(assignment, index) in task.assignments">
                    <div class="form-row"><label for="horizPass" class="col-sm-3 col-form-label"></label>
                        <div class="col-sm-4">
                            <i v-if="assignment.fileAllowed=='sourcecode'" class="fa fa-code bg-primary p-1 mr-1 float-left"></i>
                            <i v-else-if="assignment.fileAllowed=='document'" class="fa fa-book bg-primary p-1 mr-1 float-left"></i>
                            <i v-else-if="assignment.fileAllowed=='text'" class="fa fa-edit bg-primary p-1 mr-1 float-left"></i>
                            <span style="margin-left:20px">{{ assignment.description }}</span>
                        </div>
                        <div class="col-sm-3">
                            <span>
                                <i v-if="assignment.fileAllowed=='sourcecode'" >source code (.java/.py/.php)</i>
                                <i v-else-if="assignment.fileAllowed=='document'" >document (.doc/.docx/.pdf)</i>
                                <i v-else-if="assignment.fileAllowed=='text'" >text editor</i>
                            </span>
                        </div>
                        <div class="col-sm-2" v-if="mode!='delete'">
                            <button v-on:click="removeTemporaryAssignment(index)" class="btn btn-danger btn-sm"><i class="fa fa-close"></i></button>
                        </div>
                    </div>
                </div>
            </div>
            <div class="card-footer">
                <div><button v-if="mode == 'add'" v-on:click="addTask" class="btn btn-success btn-sm"><i class="fa fa-plus-square"></i> Tambahkan</button>
                <button v-if="mode == 'edit'" v-on:click="updateTask" class="btn btn-success btn-sm"><i class="fa fa-save"></i> Simpan</button>
                <button v-if="mode == 'delete'" v-on:click="deleteTask" class="btn btn-danger btn-sm"><i class="fa fa-minus-circle"></i> Hapus</button>
                <button v-on:click="closeDiv" class="btn btn-sm" ><i class="fa fa-ban"></i> Batal</button></div>
            </div>
        </div>
      </b-modal>
    </div>
</template>

<script>
  import moment from 'moment'
  import { addTask as addTaskAPI, updateTask as updateTaskAPI, deleteTask as deleteTaskAPI } from '@/api/task'
  import { successAlert, warningAlert } from '@/utils/alert'
  export default {
    name: 'editable-task',
    component: {
    },
    props: {
      mode: null,
      classroom: null,
      practicum: null,
      selectedTask: {
        type: Object,
        default: null,
        required: false
      }
    },
    mounted () {
      if (this.selectedTask != null) {
        this.task = Object.assign({}, this.selectedTask)
        this.temp.selectedDate[0] = moment(this.selectedTask.createdDate).toDate()
        this.temp.selectedDate[1] = moment(this.selectedTask.dueDate).toDate()
      }
    },
    computed: {
    },
    data () {
      return {
        task: {
          description: '',
          title: '',
          createdDate: '',
          dueDate: '',
          id: null,
          practicum: null,
          classroom: null,
          createdBy: null,
          assignments: [],
          allowLate: false
        },
        temp: {
          selectedDate: [null, null],
          assignment: {
            fileAllowed: '',
            description: ''
          }
        },
        emitter: {
          mode: '',
          task: {},
          category: ''
        },
        modal: true
      }
    },
    methods: {
      addTemporaryAssignment () {
        this.task.assignments.push(Object.assign({}, this.temp.assignment))
        this.temp.assignment.description = ''
        this.temp.assignment.fileAllowed = ''
      },
      removeTemporaryAssignment (index) {
        this.task.assignments.splice(index, 1)
      },
      assignmentDatePicker () {
        this.task.createdDate = moment(this.temp.selectedDate[0]).format('DD-MM-YYYY HH:mm:ss')
        this.task.dueDate = moment(this.temp.selectedDate[1]).format('DD-MM-YYYY HH:mm:ss')
      },
      addTask () {
        // required validation (to be implemented)
        this.task.practicum = this.practicum
        this.task.classroom = this.classroom
        this.task.createdBy = this.$store.getters.user
        addTaskAPI(this.task)
          .then(response => {
            if (response.status === 201) {
              successAlert('Tugas berhasil ditambahkan')
              this.emitter.mode = 'add'
              this.emitter.task = response.data.object.task
              this.emitter.category = response.data.object.category
              this.$emit('changelist', this.emitter)
              this.$emit('closediv')
            }
          })
          .catch(error => {
            console.log(error)
            warningAlert('Gagal menambahkan tugas : ' + error.response.data.message)
          })
      },
      updateTask () {
        // required validation (to be implemented)
        updateTaskAPI(this.task)
          .then(response => {
            if (response.status === 200) {
              successAlert('Tugas berhasil diubah')
              this.emitter.mode = 'update'
              this.emitter.task = response.data.object.task
              this.emitter.category = response.data.object.category
              this.$emit('changelist', this.emitter)
              this.$emit('closediv')
            }
          })
          .catch(error => {
            console.log(error)
            warningAlert('Gagal menyimpan perubahan : ' + error.response.data.message)
          })
      },
      deleteTask () {
        deleteTaskAPI(this.task)
          .then(response => {
            if (response.status === 200) {
              successAlert('Tugas berhasil dihapus')
              this.emitter.mode = 'delete'
              this.emitter.task = this.task
              this.$emit('changelist', this.emitter)
              this.$emit('closediv')
            }
          })
          .catch(error => {
            console.log(error)
            warningAlert('Gagal menghapus tugas : ' + error.response.data.message)
          })
      },
      closeDiv () {
        this.$emit('closediv')
      }
    },
    destroyed () {
      this.task = {}
      this.temp = {}
      this.emitter = {}
      this.selectedTask = {}
    }
  }
</script>
