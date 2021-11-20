<template>
    <div class="container">
        <div class="row mt-5">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal" data-target="#addNew">
                        Add New   
                        <i class="fa fa-user-plus fa-fw"></i>
                        </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Date Added</th>
                      <th>Actions</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="user in users" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td>{{user.type | fUpper}}</td>
                      <td>{{user.created_at | easyDate}}</td>
                     
                      <td>
                          <a href="#" @click="editModal(user)">
                              
                              <i class="fa fa-edit blue"></i>
                          </a>
                          /
                          <a href="#" @click="deleteUser(user.id)">
                              
                              <i class="fa fa-trash red"></i>
                          </a>
                      </td>
                    </tr>
                
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div> 
        <div class="modal fade" id="addNew" tabindex="-1" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 v-show="!editmode" class="modal-title" id="addNewLabel">Add New</h5>
                    <h5 v-show="editmode" class="modal-title" id="addNewLabel">Edit User</h5>
                    <button @click="close" class="close" data-bs-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                        
                    </button>
                </div>
                <form @submit.prevent="editmode ? updateUser() : createUser() ">
                  <div class="modal-body">
                      <div class="form-group">
                          <input v-model="form.name" type="text" name="name"
                          placeholder="Name"
                          class="form-control" :class="{ 'is-invalid': form.errors.has('name')}">
                          <has-error :form="form" field="name"></has-error>
                      </div>
                      <div class="form-group">
                          <input v-model="form.email" type="email" name="email"
                          placeholder="E-mail"
                          class="form-control" :class="{ 'is-invalid': form.errors.has('email')}">
                          <has-error :form="form" field="email"></has-error>
                      </div>
                      <div class="form-group">
                          <textarea v-model="form.bio" name="bio" id="bio"
                          placeholder="User Biography (Optional)"
                          class="form-control" :class="{ 'is-invalid': form.errors.has('bio')}"></textarea>
                          <has-error :form="form" field="bio"></has-error>
                      </div>

                      <div class="form-group">
                        <select name="type" id="type" v-model="form.type" class="form-control" :class="{
                          'is-invalid': form.errors.has('type') }">
                          <option value="">Select User Role</option>
                          <option value="user">Standard User</option>
                          <option value="author">Author</option>
                          </select>
                      </div>

                      <div class="form-group">
                          <input v-model="form.password" type="password" name="password" id="password"
                          placeholder="Password"
                          class="form-control" :class="{ 'is-invalid': form.errors.has('password')}">
                          <has-error :form="form" field="password"></has-error>
                      </div>
                  </div>
                
                  <div class="modal-footer">
                      <button @click="close" type="button" class="btn btn-danger" data-bs-dismiss="modal">Close</button>
                      <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
                      <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
                  </div>
                </form>
                
                </div>
            </div>
        </div>
    </div>

    <!-- Modal -->
    
</template>

<script>
    
    export default {
        data() {
          
        return{
          editmode: false,
          users: {},
            form: new Form({
              name : '',
              email : '',
              password: '',
              type : '',
              bio : '',
              photo : '',
            })
        }
    },
    methods: {
      updateUser(){},
      editModal(user){
        this.editmode = true;
        this.form.reset();
         $('#addNew').modal('show');
         this.form.fill(user)
      },
      newModal(){
        this.editmode = false;
        this.form.reset();
         $('#addNew').modal('show');
      },
      close(){
        $('#addNew').modal('hide');
      },
      deleteUser(id){
        Swal.fire({
          title: 'Are you sure?',
          text: "You won't be able to revert this!",
          icon: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Yes, delete it!'
        }).then((result) => {

          // Ajax request
          this.form.delete('api/user/'+id).then(()=>{
            
                Swal.fire(
                  'Deleted!',
                  'Your file has been deleted.',
                  'success'
                )
                Fire.$emit('Refresh');
              }).catch(()=>{
                Swal("Failed", "Something probably went wrong please try again.","warning");
              });
          

          
        })
      },
      loadUsers(){
        axios.get("api/user").then(({data}) => (this.users = data.data ));
      },

      createUser(){
         this.$Progress.start();
         this.form.post('api/user')
         .then(()=>{
           Fire.$emit('Refresh');
            $('#addNew').modal('hide');
            Toast.fire({
                icon: 'success',
                title: 'User created successfully'
                })
            this.$Progress.finish();
         })
         .catch(()=>{

         });
         
      }
    },
        created() {
            this.loadUsers();
            Fire.$on('Refresh',()=>{
              this.loadUsers();
            });
            // setInterval(()=>this.loadUsers(),3000); 
        }
    }
</script>
