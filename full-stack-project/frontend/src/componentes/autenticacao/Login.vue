<template>
    <v-container fluid>
        <v-layout>
            <v-flex>
                <v-layout column class="ma-3">
                    <h1 class="headline">Login</h1>
                    <v-divider class="mb-3" />
                        <div v-if="erros">
                            <Erros :erros="erros" />
                        </div>
                        <v-text-field label="E-mail"
                            v-model="usuario.email" />
                        <v-text-field label="Senha"
                            v-model="usuario.senha" type="password" />
                        <v-btn color="primary" class="ml-0 mt-3"
                            @click="login">
                            Logar
                        </v-btn>
                </v-layout>
            </v-flex>
            <v-flex>
                <v-layout column class="ma-3">
                    <h1 class="headline">Resultado</h1>
                    <v-divider />
                    <template v-if="dados">
                        <v-text-field label="ID" readonly
                            v-model="dados.id" />
                        <v-text-field label="Nome" readonly
                            v-model="dados.nome" />
                        <v-text-field label="E-mail" readonly
                            v-model="dados.email" />
                        <v-text-field label="Token" readonly
                            v-model="dados.token" />
                    </template>
                </v-layout>
            </v-flex>
        </v-layout>
    </v-container>
</template>

<script>
import { mapActions } from 'vuex'
import Erros from '../comum/Erros'
import gql from 'graphql-tag'

export default {
    components: { Erros },
    data() {
        return {
            usuario: {},
            dados: null,
            erros: null
        }
    },
    computed: {
        perfis() {
            return this.dados && this.dados.perfis &&
                this.dados.perfis.map(p => p.name).join(',')
        }
    },
    methods: {
        ...mapActions(['setUsuario']),
        login() {
            this.$api.query({
                query: gql`
                    query (
                        $email: String!
                        $password: String!    
                    ) {
                        login (
                            data: {
                                email: $email
                                password: $password
                            }
                        ) {
                            id name email token profiles { name }
                        }
                    }
                `,
                variables: {
                    email: this.usuario.email,
                    password: this.usuario.senha
                }
            }).then(result => {
                this.dados = {
                    ...result.data.login,
                    nome: result.data.login.name,
                    perfis: result.data.login.profiles
                }
                this.erros = null
                this.setUsuario(this.dados)
            }).catch(error => {
                this.setUsuario(null)
                this.erros = error
            })
        }
    }
}
</script>

<style>

</style>
