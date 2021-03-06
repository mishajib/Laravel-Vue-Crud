<template>
    <v-data-table
            item-key="name"
            class="elevation-1"
            :headers="headers"
            :items="roles"
            sort-by="name"
            color="error"
            :loading="loading"
            loading-text="Loading... Please wait"
    >
        <template v-slot:top>
            <v-toolbar flat color="dark">
                <v-toolbar-title>Role Management System</v-toolbar-title>
                <v-divider
                        class="mx-4"
                        inset
                        vertical
                ></v-divider>
                <v-spacer></v-spacer>
                <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ on }">
                        <v-btn color="error" dark class="mb-2" v-on="on">Add New Role</v-btn>
                    </template>
                    <v-card>
                        <v-card-title>
                            <span class="headline">{{ formTitle }}</span>
                        </v-card-title>

                        <v-card-text>
                            <v-container>
                                <v-row>
                                    <v-col cols="12" sm="12">
                                        <v-text-field color="error" v-model="editedItem.name"
                                                      label="Role Name"></v-text-field>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-card-text>

                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="error darken-1" text @click="close">Cancel</v-btn>
                            <v-btn color="error darken-1" text @click="save">Save</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-toolbar>
        </template>
        <template v-slot:item.action="{ item }">
            <v-icon
                    small
                    class="mr-2"
                    @click="editItem(item)"
            >
                mdi-content-save-edit-outline
            </v-icon>
            <v-icon
                    small
                    @click="deleteItem(item)"
            >
                mdi-delete
            </v-icon>
        </template>
        <template v-slot:no-data>
            <v-btn color="error" @click="initialize">Reset</v-btn>
        </template>
        <v-snackbar
                v-model="snackbar"
        >
            Record deleted successfully...
            <v-btn
                    color="error"
                    text
                    @click="snackbar = false"
            >
                Close
            </v-btn>
        </v-snackbar>
    </v-data-table>
</template>

<script>
    export default {
        data: () => ({
            dialog: false,
            loading: false,
            snackbar: false,
            headers: [
                {
                    text: '#',
                    align: 'left',
                    sortable: false,
                    value: 'id',
                },
                {text: 'Name', value: 'name'},
                {text: 'Created At', value: 'created_at'},
                {text: 'Updated At', value: 'updated_at'},
                {text: 'Actions', value: 'action', sortable: false},
            ],
            roles: [],
            editedIndex: -1,
            editedItem: {
                id: '',
                name: '',
                created_at: '',
                updated_at: '',
            },
            defaultItem: {
                id: '',
                name: '',
                created_at: '',
                updated_at: '',
            },
        }),

        computed: {
            formTitle() {
                return this.editedIndex === -1 ? 'ADD New Role' : 'Edit Role';
            },
        },

        watch: {
            dialog(val) {
                val || this.close();
            },
        },

        created() {
            this.initialize();
        },

        methods: {
            initialize() {
                // Add a request interceptor
                axios.interceptors.request.use((config) => {
                    this.loading = true;
                    return config;
                }, (error) => {
                    this.loading = false;
                    return Promise.reject(error);
                });

                // Add a response interceptor
                axios.interceptors.response.use((response) => {
                    this.loading = false;
                    return response;
                }, (error) => {
                    this.loading = false;
                    return Promise.reject(error);
                });
                axios.get('/api/roles', {})
                    .then((result) => {
                        this.roles = result.data.roles;
                        // this.roles.push(result.data.roles[0]);
                    }).catch((err) => {
                    if (err.response.status === 401) {
                        localStorage.removeItem('token');
                        this.$router.push('/login');
                    }
                });
            },

            editItem(item) {
                this.editedIndex = this.roles.indexOf(item);
                this.editedItem = Object.assign({}, item);
                this.dialog = true;
            },

            deleteItem(item) {
                const index = this.roles.indexOf(item);
                let decide = confirm('Are you sure you want to delete this item?');
                if (decide) {
                    axios.delete('/api/roles/' + item.id)
                        .then((result) => {
                            this.snackbar = true;
                            this.roles.splice(index, 1);
                        }).catch((err) => {
                        console.log(err.response);
                    });
                }
            },

            close() {
                this.dialog = false;
                setTimeout(() => {
                    this.editedItem = Object.assign({}, this.defaultItem);
                    this.editedIndex = -1;
                }, 300)
            },

            save() {
                if (this.editedIndex > -1) {
                    axios.put('/api/roles/'+this.editedItem.id, {'name': this.editedItem.name})
                        .then((result) => {
                            Object.assign(this.roles[this.editedIndex], result.data.role);
                        }).catch((err) => {
                            console.dir(err.response);
                    });
                    //Object.assign(this.roles[this.editedIndex], this.editedItem);
                } else {
                    axios.post('/api/roles', {'name': this.editedItem.name})
                        .then((result) => {
                            this.roles.push(result.data.role);
                        }).catch((err) => {
                        console.dir(err.response);
                    });
                }
                this.close();
            },
        },
    }
</script>

<style scoped></style>