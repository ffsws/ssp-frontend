<template>
    <div>
        <div class="hero is-light">
            <div class="hero-body">
                <div class="container">
                    <h1 class="title"><i class="material-icons">account_circle</i> Docker Pull-Secret anlegen</h1>
                </div>
                <h2 class="subtitle">
                    Hier kannst du deinem Projekt Pull-Zugriff geben auf ein Docker Repository</h2>
            </div>
        </div>
        <br>
        <form v-on:submit.prevent="createPullSecret">
            <b-field label="Openshift Projekt"
                     :type="errors.has('Openshift Projekt') ? 'is-danger' : ''"
                     :message="errors.first('Openshift Projekt')">
                <b-input v-model.trim="project"
                         placeholder="projekt-dev"
                         name="Projekt-Name"
                         ref="autofocus"
                         v-validate="'required'">
                </b-input>
            </b-field>

            <b-field label="Docker Repository"
                     :type="errors.has('Docker Repository') ? 'is-danger' : ''"
                     :message="errors.first('Docker Repository')">
                <b-input v-model.trim="repository" name="repository"></b-input>
            </b-field>

            <b-field label="Benutzername"
                     :type="errors.has('Benutzername') ? 'is-danger' : ''"
                     :message="errors.first('Benutzername')">
                <b-input v-model.trim="username" name="username"></b-input>
            </b-field>

            <b-field label="Passwort"
                     :type="errors.has('Passwort') ? 'is-danger' : ''"
                     :message="errors.first('Passwort')">
                <b-input v-model.trim="password" name="password"></b-input>
            </b-field>

            <button :disabled="errors.any()"
                    v-bind:class="{'is-loading': loading}"
                    class="button is-primary">Secret erstellen
            </button>
        </form>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                project: '',
                repository: '',
                username: '',
                password: '',
                loading: false,
            };
        },
        methods: {
            createPullSecret: function () {
                this.$validator.validateAll().then((result) => {
                    if (result) {
                        this.loading = true;

                        this.$http.post(this.$store.state.backendURL + '/api/ose/secret/pull', {
                            project: this.project,
                            repository: this.repository,
                            username: this.username,
                            password: this.password
                        }).then(() => {
                            this.loading = false;
                        }, () => {
                            this.loading = false;
                        });
                    }
                });
            }
        }
    };
</script>
