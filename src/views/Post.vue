<template>
  <v-container fluid grid-list-md v-if="!preLoad">
    <v-card>
      <v-layout row wrap justify-center>
        <v-flex xs12 md6>
          <v-form refs="form">
            <v-card-text>
              <p class="display-1">
                Dados do animal
              </p>
              <v-select
                :error-messages="errors.collect('status')"
                :items="domains.status"
                data-vv-as="categoria"
                data-vv-name="status"
                label="Categoria"
                placeholder="Selecione a categoria do pet"
                v-model="pet.status"
                v-validate="'required'"
              ></v-select>
              <v-layout row wrap>
                <v-flex xs12 sm6>
                  <v-select
                    :error-messages="errors.collect('specie')"
                    :items="domains.specie"
                    data-vv-as="espécie"
                    data-vv-name="specie"
                    label="Espécie"
                    placeholder="Selecione a espécie do animal"
                    v-model="pet.especie"
                    v-validate="'required'"
                  ></v-select>
                </v-flex>
                <v-flex xs12 sm6>
                  <v-select
                    :error-messages="errors.collect('size')"
                    :items="domains.size"
                    data-vv-as="porte"
                    data-vv-name="size"
                    label="Porte"
                    placeholder="Selecione o porte do animal"
                    v-model="pet.porte"
                    v-validate="'required'"
                  ></v-select>
                </v-flex>
              </v-layout>
              <v-layout row wrap>
                <v-flex xs12 sm6>
                  <v-select
                    :error-messages="errors.collect('age')"
                    :items="domains.age"
                    data-vv-as="idade"
                    data-vv-name="age"
                    label="Idade"
                    placeholder="Selecione a idade do animal"
                    v-model="pet.idade"
                    v-validate="'required'"
                  ></v-select>
                </v-flex>
                <v-flex xs12 sm6>
                  <div>
                    <label class="caption label">Sexo</label>
                    <gender-selection v-model="pet.sexo"/>
                  </div>
                </v-flex>
              </v-layout>
              <v-layout row v-if="pet.status != 'PARA_ADOCAO'" wrap>
                <v-flex xs12 sm6>
                  <v-text-field
                    :counter="20"
                    :error-messages="errors.collect('name')"
                    data-vv-as="nome"
                    data-vv-name="name"
                    id="name"
                    label="Nome"
                    placeholder="Nome do animal"
                    v-model="pet.nome"
                    v-validate="'required|max:20'"
                  ></v-text-field>
                </v-flex>
                <v-flex xs12 sm6>
                  <v-menu
                    :close-on-content-click="false"
                    :nudge-right="40"
                    full-width
                    lazy
                    min-width="290px"
                    offset-y
                    transition="scale-transition"
                    v-model="datePicker"
                  >
                    <v-text-field
                      :error-messages="errors.collect('date')"
                      :label="dateLabel"
                      data-vv-as="data"
                      data-vv-name="date"
                      placeholder="Informe a data"
                      prepend-icon="event"
                      readonly
                      slot="activator"
                      v-model="formatDate"
                      v-validate="'required'"
                    ></v-text-field>
                    <v-date-picker
                      @input="datePicker = false"
                      locale="pt-br"
                      v-model="pet.dataAchado"
                    ></v-date-picker>
                  </v-menu>
                </v-flex>
              </v-layout>
              <v-textarea
                :counter="500"
                :error-messages="errors.collect('description')"
                data-vv-as="descrição"
                data-vv-name="description"
                label="Descrição"
                no-resize
                placeholder="Descreva o animal em poucas palavras para que possam reconhecê-lo"
                v-model="pet.descricao"
                v-validate="'required|max:500'"
              ></v-textarea>
              <div class="mt-1">
                <label class="caption label">Fotos</label>
                <image-upload
                  v-model="pet.fotos"
                />
              </div>
            </v-card-text>
          </v-form>
        </v-flex>
        <v-flex xs12 md6 v-if="pet.status != 'PARA_ADOCAO'">
          <v-card-text>
            <p class="display-1">
              Localização
            </p>
            <google-map v-model="pet.localizacao"/>
          </v-card-text>
        </v-flex>
      </v-layout>
      <v-card-actions>
        <v-btn
          :loading="loading"
          @click="submit"
          block
          color="info"
        >
          Salvar Pet
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script>
import Pets from '@/services/pets';
import GenderSelection from '@/components/GenderSelection.vue';
import GoogleMap from '@/components/GoogleMap.vue';
import ImageUpload from '@/components/ImageUpload.vue';
import domains from '@/domains';

export default {
  name: 'Post',
  components: {
    GenderSelection,
    GoogleMap,
    ImageUpload,
  },
  data() {
    return {
      domains,
      datePicker: false,
      loading: false,
      preLoad: true,
      editing: false,
      pet: {
        ativo: true,
        dataAchado: null,
        dataRegistro: '2010-10-10',
        descricao: null,
        especie: null,
        fotos: [],
        idade: null,
        localizacao: null,
        nome: null,
        porte: null,
        sexo: 'MACHO',
        status: null,
        usuario: null,
      },
    };
  },
  created() {
    const petId = this.$route.params.id;

    if (petId) {
      if (this.loggedIn) {
        Pets.getById(petId).then((response) => {
          if (this.$store.state.loggedUser.id === response.data.usuarioId) {
            this.pet = response.data;
            this.preLoad = false;
            this.editing = true;
          } else {
            this.$router.push({ name: 'search' });
            this.$toast.error('Você não possui autorização para acessar essa página!');
          }
        });
      } else {
        this.$router.push({ name: 'search' });
        this.$toast.error('Você não possui autorização para acessar essa página!');
      }
    } else {
      this.preLoad = false;
    }
  },
  methods: {
    submit() {
      this.$validator.validateAll().then((result) => {
        if (result) {
          if (!this.pet.localizacao) {
            this.$toast.error('Pedimos que informe a localização do pet para que outros usuários possam encontrá-lo.');

            return;
          }
          if (this.$store.state.loggedIn) {
            this.pet.usuarioId = this.$store.state.loggedUser.id;

            this.loading = true;
            Pets.save(this.pet).then((response) => {
              this.$toast.success('Pet cadastrado com sucesso!');
              this.$router.push({
                name: 'petDetail',
                params: {
                  id: response.data.id,
                },
              });
            }).finally(() => {
              this.loading = false;
            });
          } else {
            this.$toast.error('Por favor realize o login em sua conta para prosseguir com o cadastro do pet.');
          }
        }
      });
    },
  },
  computed: {
    loggedIn() {
      return this.$store.state.loggedIn;
    },
    formatDate() {
      if (!this.pet.dataAchado) return null;

      const [year, month, day] = this.pet.dataAchado.split('-');

      return `${day}/${month}/${year}`;
    },
    dateLabel() {
      if (this.pet.status === 'PROCURANDO_DONO') {
        return 'Achado em';
      } else if (this.pet.status === 'PROCURANDO_PET') {
        return 'Perdido em';
      } else if (this.pet.status === 'PARA_ADOTAR') {
        return 'Nascido em';
      }

      return 'Data';
    },
  },
  watch: {
    loggedIn(value) {
      if (!value && this.editing) {
        this.$router.push({ name: 'search' });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
  .label {
    color: rgba(0,0,0,0.54);
  }
</style>
