<template>
  <v-col xs="10" sm="10" md="8" lg="8" xl="8" class="mx-auto" align="center">
    <v-card class="mt-10">
      <v-list-item class="red" align="start" hover>
        <router-link title="Profil" aria-label="Aller à son profil" :to="`/Profile/${publication.userId}`">
          <v-list-item-avatar outlined color="grey darken-3">
            <v-img :src="publication.avatar" alt="photo de profil"></v-img>
          </v-list-item-avatar>
        </router-link>
        <v-list-item-content>
          <v-list-item-title class="admin font-weight-medium" v-if="publication.isAdmin == 1">ADMIN</v-list-item-title>
          <v-list-item-title class="modo font-weight-medium" v-if="publication.isAdmin == 2">MODERATEUR</v-list-item-title>
          <v-list-item-title class="font-weight-medium">{{ publication.username }}</v-list-item-title>
          <v-list-item-title class="text-caption">{{ publication.createdAt | formatDate }}</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
      <v-row>
        <v-col>
          <v-card-text class="text-start">{{ publication.content }}</v-card-text>
          <v-img contain max-height="300" :src="publication.attachment"></v-img>
        </v-col>
      </v-row>
      <v-card-actions>
        <v-col>
          <v-tooltip top v-if="publication.userId == $store.state.userId">
            <template v-slot:activator="{ on, attrs }">
              <v-btn title="Modification" aria-label="Modifier la publication" class="mr-5" v-bind="attrs" v-on="on" text small>
                <router-link aria-label="Aller à la modification" :to="`/updatePublication/${publication.id}`">
                  <v-icon color="cyan darken-2" size="1.5rem">mdi-pen-plus</v-icon>
                </router-link>
              </v-btn>
            </template>
            <i>Modifier</i>
          </v-tooltip>
          <v-tooltip top v-if="publication.userId == $store.state.userId || $store.state.isAdmin == 1 || $store.state.isAdmin == 2">
            <template v-slot:activator="{ on, attrs }">
              <v-btn title="Supprimer" aria-label="Supprimer" @click.stop="dialog = true" v-bind="attrs" v-on="on" text color="deep-orange darken-3" small>
                <v-icon size="1.5rem">mdi-delete</v-icon>
              </v-btn>
            </template>
            <i class="mt-5">Supprimer</i>
          </v-tooltip>
            <v-dialog v-model="dialog" max-width="500">
              <v-card>
                <v-card-title class="justify-center">Supprimer la publication</v-card-title>
                <v-card-actions class="justify-center" @click="dialog = false">
                  <v-btn color="red darken-1" text>Annuler</v-btn>
                  <v-btn color="red darken-3" text @click="deletePublication">Confirmer</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
        </v-col>
        <v-col>
          <v-tooltip top v-if="!isLiked">
            <template v-slot:activator="{ on, attrs }">
              <v-btn title="Like" aria-label="Aimer cette publication" icon v-bind="attrs" v-on="on" @click="addLike">
                <v-icon size="1.5rem" color="yellow">mdi-thumb-up-outline</v-icon>
              </v-btn>
            </template>
            <div :key="index" v-for="(like, index) in Likes" class="card">
              <v-list-item-avatar outlined color="grey darken-3">
                <v-img :src="like.avatar" alt="photo de profil"></v-img>
              </v-list-item-avatar>
              <strong>{{ like.username }}</strong>
            </div>
            <i>J'aime</i>
          </v-tooltip>
          <v-tooltip top v-else-if="isLiked">
            <template v-slot:activator="{ on, attrs }">
              <v-btn title="Like" aria-label="Ne plus aimer" icon v-bind="attrs" v-on="on" @click="deleteLike">
                <v-icon size="1.5rem" color="yellow">mdi-thumb-up</v-icon>
              </v-btn>
            </template>
            <div :key="index" v-for="(like, index) in Likes" class="card">
              <v-list-item-avatar outlined color="blue darken-3">
                <v-img :src="like.avatar" alt="photo de profil"></v-img>
              </v-list-item-avatar>
              <strong>{{ like.username }}</strong>
            </div>
            <i>Je n'aime plus</i>
          </v-tooltip>
          <span>{{ Likes.length }}</span>
        </v-col>
      </v-card-actions>
    </v-card>
    <v-card class="mt-10">
      <v-container>
        <v-card flat width="45vw">
          <div>
            <v-form ref="form" v-model="valid" @submit.prevent="createComment" v-on:getAllComments="mounted()">
              <v-textarea label="Créer mon commentaire" outlined v-model="comment" type="text" placeholder="Votre commentaire..." required :rules="commentRules"></v-textarea>
              <div align="center">
                <v-btn type="submit" small value="submit" color="red darken-2" dark :disabled="!valid">Poster</v-btn>
              </div>
            </v-form>
          </div>
        </v-card>
      </v-container>
    </v-card>
    <v-card :key="index" v-for="(comment, index) in commentList" class="px-5 py-5 my-5" align="start">
      <strong>Commenté par {{ comment.username }} le {{ comment.createdAt | formatDate }} </strong>
      <v-card-text class="text-start">{{ comment.content }}</v-card-text>
      <v-card-actions align="start">
          <v-col>
            <v-tooltip top v-if="comment.userId == $store.state.userId || $store.state.isAdmin == 1 || $store.state.isAdmin == 2">
              <template v-slot:activator="{ on, attrs }">
                <v-btn aria-label="Supprimer commentaire" @click="deleteComment(comment)" v-bind="attrs" v-on="on" text color="deep-orange darken-3" small>
                  <v-icon size="1.5rem">mdi-delete</v-icon>
                </v-btn>
              </template>
              <i class="mt-5">Supprimer</i>
            </v-tooltip>
          </v-col>
      </v-card-actions>
    </v-card>
  </v-col>
</template>

<script>
import axios from "axios";
import $store from "@/store/index";
import dayjs from "dayjs";
import { mapState } from "vuex";
import { PublicationService } from "./publication.service"

export default {
  name: "onePublication",
  components: {},
  data() {
    return {
      publication: {
        Comments: [],
      },
      comment: "",
      commentList: [],
      Likes: [],
      isLiked: 0,
      dialog: false,
      valid: false,
      commentRules: [
        (v) =>
          (v && v.length >= 3) ||
          "Votre commentaire doit contenir au moins 3 caractères.",
      ],
    };
  },
  computed: {
    ...mapState(["isAdmin", "userId"]),
  },
  created() {
    this.getLikes();
    PublicationService.getOnePublication(this.$route.params.id, $store.state.token)
    .then((publication) => {
      this.publication = publication
    })
    .catch((error) => {
        console.log(error);
    });   
  },
  mounted() {
		PublicationService.getAllComments(this.$route.params.id, $store.state.token)
			.then((comment) => {
				//console.log("Tableau" , publication)
				this.commentList = comment;
			})
			.catch((error) => {
				console.log(error);
			});
	},
  methods: {
    deletePublication(publication) {
      this.dialog = false;
      axios
        .delete(
          "http://localhost:5000/api/deletePublication/" + this.$route.params.id,
          {
            headers: {
              Authorization: `Bearer ${$store.state.token}`,
            },
            data: {
              userId: publication.userId
            },
          }
        )
        .then(() => {
          this.$store.dispatch("setSnackbar", {
              text: "La publication a été supprimé"
          })
          this.$router.push({
            name: "allPublications",
          });
        })
        .catch(() => {
          this.$store.dispatch("setSnackbar", {
            color: "error",
            text: "Impossible de supprimer la publication.",
          });
        });
    },
    deleteComment(comment) {
      axios
        .delete(
          `http://localhost:5000/api/deleteComment/${comment.id}`,
          {
            headers: {
              Authorization: `Bearer ${$store.state.token}`,
            },
            data: {
              userId: comment.userId
            },
          }
        )
        .then(() => {
          const index = this.commentList.findIndex((value) => value.id === comment.id)
          this.commentList.splice(index, 1)
          if (comment.userId != $store.state.userId) {
            this.$store.dispatch("setSnackbar", {
              text: "Son commentaire a été supprimé"
            })
          }
          else {
            this.$store.dispatch("setSnackbar", {
              text: "Votre commentaire a été supprimé"
            })
          }
        })
        .catch(() => {
          this.$store.dispatch("setSnackbar", {
            color: "error",
            text: "Impossible de supprimer le commentaire.",
          });
        });
    },
    deleteLike() {
      console.log("DELETE", this.isLiked)
      if (this.isLiked == 1) {
        let id = null;
        for (let i = 0; i < this.Likes.length; i++) {
            if (this.Likes[i].userId == $store.state.userId) {
              id = this.Likes[i].id;
              break;
            }
          }
        if(id) {
        axios
          .delete(
            "http://localhost:5000/api/deleteLike/" +
            id,
            {
              headers: {
                Authorization: `Bearer ${$store.state.token}`,
              },
              data: {
                userId: $store.state.userId
              }
            }
          )
          .then((response) => {
            this.isLiked = 0;
            console.log(response.data)
            this.$store.dispatch("setSnackbar", {
              text: "Like supprimé.",
            });
            this.$router.go();
          })
          .catch(() => {
            this.$store.dispatch("setSnackbar", {
              color: "error",
              text: "Veuillez réessayer.",
            });
          });
        }
        else {
          console.log("id inconnu")
        }
      }
    },
    createComment() {
      console.log($store.state)
      if (this.$refs.form.validate()) {
        axios
          .post(
            "http://localhost:5000/api/onePublication/" +
              this.$route.params.id +
              "/createComment",
            { content: this.comment,
              userId: $store.state.userId 
            },
            {
              headers: {
                Authorization: `Bearer ${$store.state.token}`,
              },
            }
          )
          .then((response) => {
            this.mounted();
            this.$store.dispatch("setSnackbar", {
              text: "Commentaire ajouté.",
            });
            this.comment = "";
            console.log("createComment", response)
          })
          .catch(() => {
            this.$store.dispatch("setSnackbar", {
              text: "Impossible d'ajouter votre commentaire.",
            });
          });
      }
    },
    mounted() {
      PublicationService.getAllComments(this.$route.params.id, $store.state.token)
        .then((comment) => {
          //console.log("Tableau" , publication)
          this.commentList = comment;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getLikes() {
      axios
        .get(
          "http://localhost:5000/api/onePublication/" +
            +this.$route.params.id +
            "/getAllLikes",
          {
            headers: {
              Authorization: `Bearer ${$store.state.token}`,
            },
          }
        )
        .then((response) => {
          this.Likes = response.data;

          for (let i = 0; i < this.Likes.length; i++) {
            if (this.Likes[i].userId == $store.state.userId) {
              this.isLiked++;
              break;
            }
          }
        })
        .catch(() => {
          this.$store.dispatch("setSnackbar", {
            color: "error",
            text: "Erreur de chargement. Veuillez réessayer.",
          });
        });
    },
    addLike() {
      if (this.isLiked == 0) {
        const data = {
          userId: $store.state.userId,
          publicationId: this.$route.params.id,
        };
        axios
          .post(
            "http://localhost:5000/api/onePublication/" +
              +this.$route.params.id +
              "/addLike",
            data,
            {
              headers: {
                Authorization: `Bearer ${$store.state.token}`,
              },
            }
          )
          .then((response) => {
            this.like = response.data;
            this.$store.dispatch("setSnackbar", {
              text: "Like ajouté !",
            });
            this.$router.go();
          })
          .catch((err) => {
            console.log(err);
          });
      } else {
        this.$store.dispatch("setSnackbar", {
          color: "error",
          text: "Veuillez réessayer.",
        });
      }
    },
    getAllComments() {
		PublicationService.getAllComments(this.$route.params.id, $store.state.token)
			.then((comment) => {
				//console.log("Tableau" , publication)
				this.commentList = comment;
			})
			.catch((error) => {
				console.log(error);
			});
	},
  },
  filters: {
    formatDate(value) {
      if (value) {
        return dayjs(String(value)).format("DD-MMM-YYYY à HH:mm");
      }
    },
  },
};
</script>

<style scoped>

a {
  text-decoration: none;
}

.container {
  padding: 5vw 0vw 2vw 0vw;
}

.admin {
  color: #26172B;
}

.modo {
  color: #001438;
}

</style>
