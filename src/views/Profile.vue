<template>
  <div class="profile" style="overflow: hidden">
    <v-app id="inspire">
      <v-main>
        <v-container>
          <v-card class="mx-auto" max-width="85%" tile>
            <v-img
              height="200"
              src="https://cdn.vuetifyjs.com/images/cards/server-room.jpg"
            ></v-img>
            <v-row>
              <!-- Style for the v-row -->
              <!-- style="margin: 2.5%; position: absolute; top: 130px"> -->
              <v-list-item>
                <v-list-item-avatar size="100">
                  <v-avatar class="ma-md-15 mx-md-15" color="red" size="164">
                    <span class="white--text headline">CJ</span>
                  </v-avatar>
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title class="title" style="margin-top: 10px">{{ name }}</v-list-item-title>
                  <v-list-item-subtitle v-if="userType == 'Citizen' || userType == 'Service Provider'" 
                    >Points: {{ points }}</v-list-item-subtitle
                  >
                  <v-list-item-subtitle v-if="userType == 'Citizen'"
                    ><a v-on:click="payment()">
                      Add points
                    </a></v-list-item-subtitle
                  >
                </v-list-item-content>
              </v-list-item>
            </v-row>
          </v-card>
        </v-container>
        <v-sheet class="mx-auto mt-5" max-width="80%">
          <div class="text-center">
            <h1 class="font-weight-light">Activities</h1>
          </div>
          <v-slide-group
            v-model="model"
            class="pa-2"
            active-class="success"
            show-arrows
          >
            <v-slide-item v-for="(n, index) in model" :key="index">
              <div class="ma-md-5 mx-md-15">
                <activity-card
                  class="ma-md-5 mx-md-1"
                  :description="n.data.description"
                  :cost="n.data.cost"
                  :activityName="n.data.name"
                  :eventStart="n.data.eventDateStart"
                  :eventEnd="n.data.eventDateEnd"
                  :categories="n.data.category"
                  :isProfileCard="isCitizen"
                  :isVenderCard="isVender"
                  :isAdminCard="isAdmin"
                />
              </div>
            </v-slide-item>
          </v-slide-group>
        </v-sheet>
      </v-main>
    </v-app>
  </div>
</template>

<script>
import firebase from "firebase";
import ActivityCard from "../components/ActivityCard.vue";

export default {
  name: "profile",
  components: { ActivityCard },
  data() {
    return {
      model: null,
      userType: this.$store.state.user.userType,
      isAdmin: false,
      name: this.$store.state.user.name,
      points: this.$store.state.user.points
    };
  },


  created() {
    console.log(this.$store.state);
    this.getCardsData();
  },

  methods: {
    logout: function () {
      firebase
        .auth()
        .signOut()
        .then(() => {
          this.$router.replace("login");
          alert("You are signed out");
        });
    },
    post: function () {
      this.$router.replace("/post");
      alert("Redirecting to post creation page");
    },
    getCardsData: async function () {
      const db = firebase.firestore();
      const d = [];
      if (this.$store.state.user.userType == "Citizen") {
        //this.isCitizen = true;
        let purchasedIDs;
        await db
          .collection("Citizens")
          .where("email", "==", this.$store.state.user.email)
          .get()
          .then(function (querySnapshot) {
            querySnapshot.forEach(function (doc) {
              purchasedIDs = doc.data().purchased.map((res) => {
                return res.activity.id;
              });
            });
          })
          .catch(function (error) {
            console.log("Error getting documents: ", error);
          });

        for (const id of purchasedIDs) {
          await db
            .collection("Activities")
            // .doc(id)
            .where(firebase.firestore.FieldPath.documentId(), '==', id)
            .get()
            .then(function (querySnapshot) {
              querySnapshot.forEach(function (doc) {
                console.log(doc.id, " => ", doc.data());
                d.push({ id: doc.id, data: doc.data() });
              });
            })
            .catch(function (error) {
              console.log("Error getting documents: ", error);
            });
        }
      } else if (this.$store.state.user.userType == "ServiceProvider") {
        //this.isVender = true;
        let docID = "";
        let userData = {
          name: "",
          points: "",
        };
        await db
          .collection("ServiceProviders")
          .where("email", "==", this.$store.state.user.email)
          .get()
          .then(function (querySnapshot) {
            querySnapshot.forEach(function (doc) {
              console.log(doc.id, " => ", doc.data());
              docID = doc.id;
              userData.name = doc.data().name;
              userData.points = doc.data().points;
            });
          });

        const vendor = "/ServiceProivders/" + docID;
        this.$store.state.user.name = userData.name;
        this.$store.state.user.points = userData.points;
        this.name = this.$store.state.user.name;
        this.points = this.$store.state.user.points = userData.points;

        await db
          .collection("Activities")
          .where("provider", "==", vendor)
          .get()
          .then(function (querySnapshot) {
            querySnapshot.forEach(function (doc) {
              // console.log(doc.id, " => ", doc.data());
              d.push({ id: doc.id, data: doc.data() });
            });
          })
          .catch(function (error) {
            console.log("Error getting documents: ", error);
          });
      } else if (this.$store.state.user.userType == "Admin") {
        this.isAdmin = true;
        await db
          .collection("Activities")
          .where("adminApproved", "==", false)
          .get()
          .then(function (querySnapshot) {
            querySnapshot.forEach(function (doc) {
              d.push({ id: doc.id, data: doc.data() });
            });
          })
          .catch(function (error) {
            console.log("Error getting documents: ", error);
          });
      }

      this.$store.activities = d;
      this.model = this.$store.activities;
    },

    payment: function () {
      this.$router.replace("/addpoints");
    },
  },
};
</script>

<style scoped>
</style>