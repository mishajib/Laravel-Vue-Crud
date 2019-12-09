<template>
    <v-app id="inspire">
        <v-content>
            <v-container
                    class="fill-height"
                    fluid
            >
                <v-row
                        align="center"
                        justify="center"
                >
                    <v-col
                            cols="12"
                            sm="8"
                            md="4"
                    >
                        <v-card class="elevation-12">
                            <v-toolbar
                                    color="error"
                                    dark
                                    flat
                            >
                                <v-toolbar-title>Login form</v-toolbar-title>
                                <v-spacer/>
                            </v-toolbar>
                            <v-card-text>
                                <v-progress-linear
                                        :active="loading"
                                        :indeterminate="loading"
                                        absolute
                                        top
                                        color="white accent-4"
                                />
                                <v-form ref="form"
                                        v-model="valid" 
                                        >
                                    <v-text-field color="error"
                                            label="Login"
                                            name="login"
                                            v-model="email"
                                            :rules="emailRules"
                                            required
                                            prepend-icon="mdi-account-circle-outline"
                                            type="text"
                                    />

                                    <v-text-field color="error"
                                            id="password"
                                            label="Password"
                                            v-model="password"
                                            name="password"
                                            prepend-icon="mdi-account-lock-outline"
                                            type="password"
                                            required
                                    />
                                </v-form>
                            </v-card-text>
                            <v-card-actions>
                                <v-spacer/>
                                <v-btn color="error" :disabled="!valid" @click="login">Login</v-btn>
                            </v-card-actions>
                        </v-card>
                            <v-snackbar
                            v-model="snackbar"
                            >
                            {{ text }}
                            <v-btn
                                color="pink"
                                text
                                @click="snackbar = false"
                            >
                                Close
                            </v-btn>
                            </v-snackbar>
                    </v-col>
                </v-row>
            </v-container>
        </v-content>
    </v-app>
</template>

<script>
    export default {
        name: "LoginComponent",
        props: {
            source: String,
        },
        data() {
            return {
                valid: true,
                email: '',
                emailRules: [
                    v => !!v || 'E-mail is required',
                    v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
                ],
                password: '',
                passwordRules: [
                    v => !!v || 'Password is required',
                ],
                loading: false,
                snackbar: false,
                text: '',
            }
        },
        methods: {
            login() {
                // Add a request interceptor
                axios.interceptors.request.use((config) => {
                    this.loading = true;
                    return config;
                }, (error) => {
                    this.loading = false;
                    return Promise.reject(error);
                });

                // Add a response interceptor
                axios.interceptors.response.use( (response) => {
                    this.loading = false;
                    return response;
                }, (error) => {
                    this.loading = false;
                    return Promise.reject(error);
                });
                axios.post('/api/login', {'email': this.email, 'password': this.password})
                .then((result) => {
                    localStorage.setItem('token', result.data.token);
                    localStorage.setItem('loggedin', true);
                    this.$router.push('/admin')
                    .then(res => console.log('LoggedIn Successfully...'))
                    .catch(err => console.log(err));
                }).catch((err) => {
                    this.text = err.response.data.status;
                    this.snackbar = true;
                });
                
            },
        },
        created() {
            this.$vuetify.theme.dark = true;
        },
    }
</script>

<style scoped>

</style>