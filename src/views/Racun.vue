<template>
  <div class="p-0 m-0 pt-16">
    <div class="grid auto-rows-auto gap-4">
      <div class="grid auto-rows-auto gap-4 pl-4 pr-4">
        <!--===================IME I PREZIME=======================-->
        <div>
          <p class="text-left text-18px m-0 p-0">Ime i prezime:</p>
          <input
            type="text"
            class="border rounded"
            id="nameInput"
            v-model="imePrezime"
          />
          <hr />
        </div>
        <!--===================/IME I PREZIME===================-->
        <!--===================EMAIL=======================-->
        <div>
          <p class="text-left text-18px m-0 p-0">Email</p>
          <input type="email" id="emailInput" v-model="email" />
          <hr />
          <h2
            v-if="!checkEmail(email) && email"
            class="CWarning"
            id="resultEmail"
          >
            Email nije točno napisan!
          </h2>
        </div>
        <!--===================EMAIL END===================-->
        <!--===================OIB=========================-->
        <div>
          <p class="text-left text-18px m-0 p-0">OIB</p>
          <input
            type="text"
            id="oibInput"
            v-model="oib"
            oninput="this.value = this.value.replace(/[^0-9.]/g, '').replace(/(\..*?)\..*/g, '$1');"
            maxlength="11"
          />
          <hr />
          <h2 v-if="oib.length != 11 && oib" class="CWarning" id="resultOIB">
            OIB mora sadržavati 11 brojeva!
          </h2>
        </div>
        <!--===================OIB END=====================-->
        <!--===================BROJ MOBITELA===============-->
        <div>
          <p class="text-left text-18px m-0 p-0">Broj mobitela</p>
          <div class="flex">
            <div class="Cmt-3" style="color: grey !important">09</div>
            <div class="w-full">
              <input
                type="text"
                name="mob"
                id="mobInput"
                v-model="UpdateMob"
                oninput="this.value = this.value.replace(/[^0-9.]/g, '').replace(/(\..*?)\..*/g, '$1');"
                maxlength="10"
              />
              <hr />
            </div>
          </div>
          <h2
            v-if="UpdateMob.length != 9 && UpdateMob.length != 10 && UpdateMob"
            class="CWarning"
            id="resultMob"
          >
            Broj mobitela mora sadržavati 10 ili 11 brojeva!
          </h2>
        </div>
        <!--===================BROJ MOBITELA END===========-->
        <!--===================IBAN===============-->
        <div v-if="store.currentUser!='istramusicshop@gmail.com'">
          <p class="text-left text-18px m-0 p-0">IBAN</p>
          <div class="flex">
            <div class="Cmt-3" style="color: grey !important">HR</div>
            <div class="w-full">
              <input
                type="text"
                name="iban"
                id="iban"
                v-model="iban"
                oninput="this.value = this.value.replace(/[^0-9.]/g, '').replace(/(\..*?)\..*/g, '$1');"
                maxlength="19"
              />
              <hr />
            </div>
          </div>
          <h2
            v-if="iban.length != 19 && iban!=''"
            class="CWarning mt-1"
            id="resultIban"
          >
            IBAN mora sadržavati 19 brojeva!
          </h2>
        </div>
        <!--===================/IBAN===========-->
        <!--===================THEME===================-->
        <div @click="updateTheme()">
          <p class="text-left text-18px m-0 p-0 mb-1">Izgled aplikacije</p>
          <CSelect
            id="$THEME"
            :options="['Svijetla', 'Tamna Plava', 'Tamna Crvena']"
            :default="theme"
            v-model="theme"
          />
        </div>
        <!--===================/THEME===================-->
        <!--==============SPREMI================-->
        <div class="place-self-center mt-12" :class="IsAllFilled ? 'active' : 'inactive'">
          <CButtonAccept btn="AŽURIRAJ PODATKE" :btnClickHandler="IsAllFilled ? updateKorisnik : dummy" />
        </div>
        <!--==============/SPREMI============-->
        <CSuccess
          :class="!canSave ? 'hide2' : 'hide'"
           msg1="Spremljeno!"
          :msg2="!emailUpdated ? 'Uneseni podaci su spremljeni. Nakon nestanka ove obavijesti možete ponovno spremiti podatke.'
          : 'Uneseni podaci su spremljeni. Odjavit ćemo Vas kako biste se prijavili novim emailom'"
        />
        <CWarning
          :class="!wrongPass ? 'hide2' : 'hide'"
           msg1="Upozorenje!"
          msg2="Unijeli ste krivu lozinku za promjenu emaila."
        />
        <!--===================RESETIRAJ LOZINKU====================-->
        <div class="place-self-center mt-6 mb-32"> 
          <router-link to="/password-reset">
            <CButtonDecline btn="RESETIRAJ LOZINKU" :btnClickHandler="dummy" />
          </router-link>
        </div>
        <!--================/RESETIRAJ LOZINKU===================-->
        
      </div>
    </div>
    <!--==============LOGUT================-->
    <button class="logout" @click="signout()">
      <p class="pr-2 pb-1 text-24px">Odjava</p>
      <img src="@/assets/exit_icon.svg" />
    </button>
    <!--==============/LOGUT================-->
  </div>
</template>
<script>
import store from "@/store";
//Komponente
import CWarning from "@/components/CWarning.vue";
import CSelect from "@/components/CSelect.vue";
import CButtonAccept from "@/components/CButtonAccept.vue";
import CButtonDecline from "@/components/CButtonDecline.vue";
import CSuccess from "@/components/CSuccess.vue";
//Firebase
import { getAuth, signOut } from "@/firebase";
import { collection, getDocs } from "@/firebase";
import { doc, updateDoc } from "@/firebase";
import { db, } from "@/firebase";
import {updateEmail, reauthenticateWithCredential, EmailAuthProvider} from "@/firebase";

function eye() {
  let x = document.getElementById("dg-input-elem");
  let y = document.getElementById("eye1");
  let z = document.getElementById("eye2");
  if (x.type === "password") {
    x.type = "text";
    y.classList.add("invisible");
    z.classList.remove("invisible");
  } else {
    x.type = "password";
    z.classList.add("invisible");
    y.classList.remove("invisible");
  }
}

function bindingFunction() {
  document.getElementById('eye1').onclick = function() {
    eye();
  };
  document.getElementById('eye2').onclick = function() {
    eye();
  };
}

let wait = function (seconds) {
  return new Promise((resolveFn) => {
    setTimeout(resolveFn, seconds * 1000);
  });
};

export default {
  name: "Racun",
  data() {
    return {
      imePrezime: "",
      oldEmail: "",
      email: "",
      oib: "",
      mobTemp: "",
      mob: "",
      iban: "",
      theme: store.theme,
      canSave: true,
      wrongPass: true,
      store,
      emailUpdated: false,
    };
  },
  components: {
    CWarning,
    CButtonAccept,
    CButtonDecline,
    CSelect,
    CSuccess,
  },
  mounted() {
    this.readData();
  },
  methods: {
    async readData() {
      const querySnapshot = await getDocs(collection(db, "users"));
      querySnapshot.forEach((doc) => {
        if (store.userID === `${doc.id}`) {
          if(store.currentUser != `${doc.data().email}`)
            this.updateEmail();
          this.imePrezime = `${doc.data().imePrezime}`;
          this.oldEmail = store.currentUser;
          this.email = store.currentUser;
          this.oib = `${doc.data().oib}`;
          this.mobTemp = `${doc.data().mob}`;
          this.iban = `${doc.data().iban}`.slice(2);
          this.mobLoad();
          store.userID = `${doc.id}`;
        }
      });
    },
    //Potrebno kod poništavanja promjene preko emaila
    async updateEmail(){
      const g = doc(db, "users", store.userID);
      await updateDoc(g, {
        email: store.currentUser,
      });
    },
    async updateKorisnik() {
      if(this.email !== this.oldEmail) {
        //Password hide icon 
        setTimeout(() => {
          document.getElementsByClassName("dg-form")[0].innerHTML = ('<label for="dg-input-elem" style="font-size: 13px;">Molimo unesite vašu lozinku:</label>' +
            '<input type="password" placeholder="" autocomplete="off" id="dg-input-elem" style="width: 100%; margin-top: 10px; padding: 5px 15px; font-size: 16px; border-radius: 4px; border: 2px solid rgb(238, 238, 238);" type = "password">' +
            ' <hr/>' +
            ' <img ' +
            'id="eye1"' +
            ' class="eye"' +
            ' src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/79/OOjs_UI_icon_eyeClosed.svg/1200px-OOjs_UI_icon_eyeClosed.svg.png"/>' +
            ' <img ' +
            'id="eye2"' +
            ' class="eye invisible"' +
            ' src="https://image.flaticon.com/icons/png/512/63/63568.png"/>'
            );
          bindingFunction();
        }, 100)
        try {
          await this.$dialog
          .prompt({
            title: "Promjena email adrese",
            }, {
            promptHelp: 'Molimo unesite vašu lozinku:'
            })
            const credential = EmailAuthProvider.credential(
            store.currentUser,
            document.getElementById("dg-input-elem").value, //Password from dialogue
            );
            const auth = getAuth();
            const user = auth.currentUser;

            await reauthenticateWithCredential(user, credential);
              await updateEmail(user, this.email);
                console.log("Email updated.");
                store.currentUser = this.email;
                this.saveData();
                this.emailUpdated = true;
                await wait(5);
                  this.signout();
          }
          catch(e) {  
            if(e == "FirebaseError: Firebase: Error (auth/wrong-password)."){ 
                this.wrongPass = false;
                setTimeout(() => {
                this.wrongPass = true;
                }, 4000);
            }
          };
      }
      else this.saveData();
    },
    //Spremi korisničke podatke u firestore
    async saveData() {
      try {
      this.canSave = false;
      const g = doc(db, "users", store.userID);
      await updateDoc(g, {
          imePrezime: this.imePrezime,
          email: this.email,
          mob: this.mob,
          oib: this.oib,
          theme: this.theme,
        })
        console.log("Podaci o korisniku spremljeni!");
        setTimeout(() => {
          this.canSave = true;
        }, 5000);
      }  
      catch(e) {
        console.error("Neuspješno spremanje podataka o korisniku!" + e);
      }
    },
    eye() {
      console.log("eye");
      let x = document.getElementById("dg-input-elem");
      let y = document.getElementById("eye1");
      let z = document.getElementById("eye2");
      if (x.type === "password") {
        x.type = "text";
        y.classList.add("invisible");
        z.classList.remove("invisible");
      } else {
        x.type = "password";
        z.classList.add("invisible");
        y.classList.remove("invisible");
      }
    },
    dummy() {},
    async signout() {
      store.theme="Svijetla"; //Theme reset
      try {
        const auth = getAuth();
        await signOut(auth);
        console.log("Signed out!");
      }
      catch(e) {
        console.error("Error signing out!" + e);
      }
    },
    checkEmail(email) {
      let re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return re.test(email);
    },
    updateTheme() {
      store.theme=this.theme;
    },
    mobLoad() {
      let br = this.mobTemp.slice(2);
      this.mob = this.mobTemp;
      let l = br.length;
      if (l >= 1 && l < 5 && br)
        this.mobTemp = br.slice(0, 1) + "-" + br.slice(1);
      else if (l >= 5)
        this.mobTemp =
        br.slice(0, 1) + "-" + br.slice(1, 4) + "-" + br.slice(4);
    },
  },
  computed: {
    IsAllFilled() {
      return this.imePrezime &&
        this.checkEmail(this.email) &&
        this.email &&
        this.oib &&
        this.oib.length == 11 &&
        this.UpdateMob.length > 8 &&
        this.iban.length == 19 &&
        this.UpdateMob &&
        this.canSave
        ? true
        : false;
    },
    UpdateMob: {
      get() {
        return `${this.mobTemp}`;
      },
      set() {
        let br = document.getElementById("mobInput").value.replace(/-/g, "");
        if (br == "") {
          this.mob = "";
          this.mobTemp = "";
        }
        this.mob = "09" + br;
        let l = br.length;
        if (l >= 1 && l < 5 && br)
          this.mobTemp = br.slice(0, 1) + "-" + br.slice(1);
        else if (l >= 5)
          this.mobTemp =
            br.slice(0, 1) + "-" + br.slice(1, 4) + "-" + br.slice(4);
      },
    },
  },
};
</script>

<style scoped lang="scss">
.logout {
  position: fixed;
  bottom: 0;
  width: 100%;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  filter: drop-shadow(0px -4px 4px rgba(0, 0, 0, 0.25));
  background: var(--FluorescentRed__FrenchWine);
  & p {
    display: flex;
    align-items: center;
    color: #000000d0;
    font-weight: bold;
  }
  & img {
    width: 32px;
    height: 32px;
    opacity: 0.75;
  }
}

.hide {
  overflow: hidden;
  height: 0px;
}
.hide2 {
  overflow: hidden;
  height: 106px;
}

.Cmt-3 {
  margin-top: 3px;
}
</style>
