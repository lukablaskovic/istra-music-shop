<template>
  <div class="p-0 m-0" :class="!mode ? 'pt-16' : 'below-ucitaj pt-16'">
    <div class="grid auto-rows-auto gap-4">
      <div class="grid auto-rows-auto gap-4 pl-4 pr-4 mt-2 mb-16">
        <!--===================VRSTA INSTRUMENTA===================-->
        <div>
          <p class="text-left text-18px m-0 p-0 mb-1">Vrsta instrumenta</p>
          <CSelect
            :options="vrste"
            :default="odabranaVrsta"
            v-model="odabranaVrsta"
          />
        </div>
        <!--===================/VRSTA INSTRUMENTA===================-->
        <!--===================PROIZVOĐAČ===================-->
        <div>
          <p class="text-left text-18px m-0 p-0 mb-1">Proizvođač</p>
          <CSelect
            :options="proizvodacSet"
            :default="odabraniProizvodac"
            v-model="odabraniProizvodac"
          />
        </div>
        <!--===================/PROIZVOĐAČ===================-->
        <!--===================MODEL===================-->
        <div>
          <p class="text-left text-18px m-0 p-0 mb-1">Model</p>
          <CSelect
            :options="modelSet"
            :default="odabraniModel"
            v-model="odabraniModel"
          />
        </div>
        <!--===================/MODEL===================-->
        <!--===================SERIJA===================-->
        <div>
          <p class="text-left text-18px m-0 p-0 mb-1">Serija</p>
          <CSelect
            :options="serijaSet"
            :default="odabranaSerija"
            v-model="odabranaSerija"
          />
        </div>
        <!--===================/SERIJA===================-->
        <!--===================GODINA PROIZVODNJE====================-->
        <div>
          <p class="text-left text-18px m-0 p-0">Godina proizvodnje</p>
          <input
            type="text"
            class="border rounded"
            id="emailInput"
            v-model="godinaProizvodnje"
            placeholder="1950..."
            maxlength="4"
          />
          <hr />
        </div>
        <h2 v-if="yearCheck" class="CWarning">
          Molimo unesite ispravnu godinu!
        </h2>
        <!--================/GODINA PROIZVODNJE===================-->
        <!--===================VLASNIK===================-->
        <div>
          <p class="text-left text-18px m-0 p-0 mb-1">Vlasnik</p>
          <CSelect
            :options="['1', '2', '3']"
            :default="vlasnik"
            v-model="vlasnik"
          />
        </div>
        <!--===================/VLASNIK===================-->
        <!--===================STANJE===================-->
        <div>
          <p class="text-left text-18px m-0 p-0 mb-1">Stanje</p>
          <CSelect
            :options="stanjaSet"
            :default="stanje"
            v-model="stanje"
          />
        </div>
        <!--===================/STANJE===================-->
        <!--===================SLIKE===================-->
        <div id="scroll-slike">
          <p class="text-left text-18px m-0 p-0">Slike</p>
          <div class="grid grid-rows-2 grid-cols-3 gap-3 mt-2" >

            <div v-for="(z, i) of instrumentSlike"
            :key="`key-${i}`">
              <p class="otkup-img-text">{{z}}</p>
              <div class="square img_1-1" :class="{'img-top-left' : i==0, 'img-top' : i==1, 'img-top-right' : i==2,
                                                    'img-bottom-left' : i==3, 'img-bottom' : i==4, 'img-bottom-right' : i==5}">
                  <div :class="!isUploadingPicture[i] ? 'hide-loading' : 'show-loading'" 
                        class="loader-squre-img">
                    <div class="loader animate-spin rounded-full border-4 border-t-4 h-6 w-6"></div>
                  </div>
                  <button :class="isUploadingPicture[i] ? 'hide-text' : 'show-text'" 
                          class="square-img2" @click="imageUpload(i)">Učitaj sliku</button> 
                  <img class="square-img" :src="pictures.guitarPictures[i].url"/>
              </div>
            </div> 
            
          </div>
        </div>
        <!--===================/SLIKE===================-->
        <!--===================/NAPOMENA===================-->
        <div>
          <p class="text-left text-18px m-0 p-0">Napomena</p>
          <textarea
            v-model="napomena"
            class="resize-none rounded-md w-full h-32 mb-4 mt-4 otkup-textarea"
          ></textarea>
        </div>
        <!--===================/NAPOMENA===================-->
        <!--===================/OTKUPI===================-->
        <div
          class="place-self-center mb-4"
          :class="allFilled ? 'active' : 'inactive'"
        >
          <CButtonSingle
            btn="POŠALJI"
            :btnClickHandler="allFilled ? otkupi : dummy"
          />
        </div>
        <!--===================/OTKUPI===================-->
      </div>
      <!--==============FOOTER==================================-->
      <div class="menu-bottom grid grid-cols-1 mt-4">
        <div class="bg-bottom"></div>
        <div class="menu-item-bottom text-left mx-auto">
          <img
            v-if="store.darkToggle"
            class="money"
            src="@/assets/money_icon.svg"
          />
          <img
            v-if="!store.darkToggle"
            class="money"
            src="@/assets/money_icon_dark.svg"
          />
          <p class="pl-4 text-24px">
            Predložena cijena:
            <b class="price">{{ izracunCijene }}</b>
          </p>
        </div>
      </div>
      <!--==============FOOTER END==============================-->
    </div>

<!--=================================IMAGE UPLOAD=========================================-->
<!--======================================================================================-->
<div :class="mode ? 'show-ucitaj' : 'hide-ucitaj'">

    <div class="CDodavanje-bottom grid grid-cols-1">
      <div class="place-self-center" :class="isSpining ? 'photo-iconD' : ''">
          <CButtonSingle
            :class="canUpload && !isSpining? '' : 'transparent-25'"
            btn="PRENESI  "
            :btnClickHandler="canUpload && !isSpining ? savePhoto : dummy"
          />
        <a href="#scroll-slike">
          <img @click="!isUploading ? (
          mode = !mode, 
          pictures.mode = !pictures.mode,
          isUploadingPicture[0] = false,
          isUploadingPicture[1] = false,
          isUploadingPicture[2] = false,
          isUploadingPicture[3] = false,
          isUploadingPicture[4] = false,
          isUploadingPicture[5] = false) : ''" 
          :class="!isUploading && !isSpining ? '' : 'hide-loading'"
          class="arrow" src="@/assets/arrow_icon.png" />
        </a>
        <div class="loading flex mt-4" :class="!isUploading ? 'hide-loading' : 'show-loading'">
          <div class="loader animate-spin rounded-full border-4 border-t-4 h-6 w-6 mr-2"></div>
          Učitavanje slike
        </div>
      </div>
      <div v-if="isSpining" class="place-self-center">
          <CButtonSingle
            :class="canUpload && !isSpining? '' : 'transparent-25'"
            btn="PRENESI  "
            :btnClickHandler="canUpload && !isSpining ? savePhoto : dummy"
          />
          <div class="loading flex mt-4" :class="!isUploading ? 'hide-loading' : 'show-loading'">
            <div class="loader animate-spin rounded-full border-4 border-t-4 h-6 w-6 mr-2"></div>
            Učitavanje slike
        </div>
      </div>
    </div>

    <div class="CDodavanje-top">
      <div v-if="uploaded()" class="mx-auto">
        <img v-if="store.theme=='Svijetla'" class="dodavanje-icon" src="@/assets/upload.svg" />
        <img v-if="store.theme=='Tamna Plava'" class="dodavanje-icon" src="@/assets/upload_dark_blue.svg" />
        <img v-if="store.theme=='Tamna Crvena'" class="dodavanje-icon" src="@/assets/upload_dark_red.svg" />
        <p>Molimo odaberite sliku za</p>
        <p>{{ uploadText }}</p>
        <p v-if="currentPictureObj.id != 4 && currentPictureObj.id != 5">
          stranu gitare
        </p>
      </div>
      
      <div v-if="!uploaded()" class="loading2 flex mt-4 show-loading">
        <div class="loader animate-spin rounded-full border-4 border-t-4 h-6 w-6 mr-2"></div>   
      </div>

        <croppa
          :class="isUploading || isSpining ? 'hide-x' : 'show-x'"
          v-if="refresh"
          id="checkForUpload"
          v-model="imageReference"
          auto-sizing
          :placeholder="''"
          :accept="'image/*'"
          :file-size-limit="0"
          :quality="2"
          :zoom-speed="3"
          :disabled="false"
          :disable-drag-and-drop="false"
          :disable-click-to-choose="false"
          :disable-drag-to-move="false"
          :disable-scroll-to-zoom="false"
          :disable-rotation="false"
          :prevent-white-space="true"
          :reverse-scroll-to-zoom="false"
          :show-remove-button="true"
          :remove-button-color="'gray'"
          @new-image="canUpload = true, dummy(), uploading = false"
          @image-remove="canUpload = false,dummy()"
        ></croppa>

      
      <div class="menu-bottom-dodavanje grid grid-cols-1 mt-4">
        <div class="mx-auto "> 
          
        </div>
      </div>
      <div class="menu-bottom-dodavanje grid grid-cols-1 mt-4">
        <div class="mx-auto" :class="isUploading || !canUpload ? 'transparent' : ''"> 
          <img @click="canUpload ? rotateImg() : dummy()" class="photo-icon" :class="isSpining ? 'photo-iconD' : ''" src="@/assets/rotate.png" />
          <img v-if="isSpining" class="photo-iconR" src="@/assets/rotate.png" />
        </div>
      </div>

    </div>
  </div>
  
<!--======================================================================================-->
<!--=================================/IMAGE UPLOAD========================================-->
  </div>
</template>
<script>
import data from "../assets/JSON/InstrumentData.json";
//Components
import CWarning from "@/components/CWarning.vue";
import CButtonAccept from "@/components/CButtonAccept.vue";
import CButtonDecline from "@/components/CButtonDecline.vue";
import CSelect from "@/components/CSelect.vue";
import CButtonSingle from "@/components/CButtonSingle.vue";
//Local storage
import store from "@/store";
import pictures from "@/pictures";
//Firebase
import { collection, setDoc, doc} from "@/firebase";
import { getDocs } from "@/firebase";
import { db } from "@/firebase";
//Picture upload
import { storage, ref, uploadBytes, getDownloadURL, deleteObject, getStorage} from "@/firebase";

import router from "@/router";
import emailjs from "@emailjs/browser";

let wait = function (seconds) {
  return new Promise((resolveFn) => {
    setTimeout(resolveFn, seconds * 1000);
  });
};

export default {
  name: "OtkupOpreme",
  data() {
    return {

      //User
      imePrezime: "",
      email: "",
      oib: "",
      mob: "",
      iban: "",
      //Page switch
      mode: false,
      //Instrument data
      jsonData: data,
      vrste: ["Gitara"],
      prikazaniProizvodaci: [],
      prikazaniModeli: [],
      prikazaneSerije: [],
      instrumentSlike: ['Prednja', 'Stražnja', 'Bočna desna', 'Bočna lijeva', 'Serijski broj', 'Glava gitare'],
      //Pomocni podaci
      prikazanaStanja: [],
      cijeneSerija: {},
      //Odabrani podaci
      odabranaVrsta: "Gitara",
      odabraniProizvodac: "Gibson",
      odabraniModel: "Les Paul",
      odabranaSerija: "Standard",
      godinaProizvodnje: "",
      vlasnik: "1",
      stanje: "Novo",
      napomena: "",
      //Calculated
      sifra : new Date().getFullYear()+"-"+Math.floor(100000 + Math.random() * 900000),
      preporucenaCijena: 0,
      zahtjevPredan: false,

      //Data
      store,
      pictures,
      
      //UcitajSliku
      imageReference: null,
      canUpload: false,
      uploadText: "",
      currentPictureObj: [null,null,null,null,null,null],
      isUploading: false,
      isUploadingPicture: [false,false,false,false,false,false],
      refresh: true,
      animateRotate: false,
      isSpining: false,
    };
  },
  components: {
    CWarning,
    CButtonAccept,
    CButtonDecline,
    CSelect,
    CButtonSingle,
  },
  async beforeRouteLeave(to, from, next) {
    //Provjeri je li doslo do promjene kod promjene rute
    if(
      !this.mode &&
      this.odabraniProizvodac === "Gibson" &&
      this.odabranaVrsta === "Gitara" &&
      this.odabraniModel === "Les Paul" &&
      this.odabranaSerija === "Standard" &&
      this.vlasnik === "1" &&
      this.stanje === "Novo" &&
      this.napomena === "" &&
      this.godinaProizvodnje === "" &&
      pictures.checkIfUploaded()) {
        pictures.resetData();
        next();
        return; 
      }
    //Ako je zahtjev predan, resetiraj slike i prebaci 
    else if(this.zahtjevPredan) {
        pictures.resetData();
        next();
        return; 
      }
    //Ako je doslo do promjene a zahtjev nije predan, upozori korisnika na gubitak podataka
    else {
      try {
        await this.$dialog.confirm ("Jeste li sigurni da želite napustiti stranicu? Promjene neće biti spremljene.");
          this.deleteImages();
          pictures.resetData();
          next();
      }
      catch(e) {
        next(false);
      }  
    }
  },
 mounted() {
    this.fetchUserData();
  },
  methods: {
    dummy() {},
    rotateImg() {
      this.isSpining = true;
      this.animateRotate=!this.animateRotate;
      setTimeout(() => {
        this.imageReference.rotate(); 
        this.animateRotate=!this.animateRotate;
        this.isSpining = false;
      }, 100)
    },
    uploaded() {
      if (this.imageReference != null)
        if (this.imageReference.chosenFile == null)
          return true;
      return false;
    },
    deleteImages(){
      const storage = getStorage();
        pictures.guitarPictures.filter(picture => picture.name!="").forEach((picture) => {
          let imagesRef = ref(storage, "zahtjevi/" + store.currentUser + "/" + this.sifra + "/" + picture.name);
          deleteObject(imagesRef);
        });
    },
    async fetchUserData() {
      const querySnapshot = await getDocs(collection(db, "users"));
      querySnapshot.forEach((doc) => {
        if (store.currentUser === `${doc.data().email}`) {
          this.imePrezime = `${doc.data().imePrezime}`;
          this.email = `${doc.data().email}`;
          this.oib = `${doc.data().oib}`;
          this.mob = `${doc.data().mob}`;
          this.iban = `${doc.data().iban}`;
        }
      });
    },
    getImage() {
      return new Promise((resolveFn, errorFn) => {
        this.imageReference.generateBlob((data) => {
          resolveFn(data);
        });
      });
    },
    //Spremi sliku u firebase storage
    async savePhoto() {
      try {
        this.canUpload = false;
        this.isUploading = true;
        
        let blobData = await this.getImage();
          let objectName = Date.now() + ".png";
          let imageName = "zahtjevi/" + store.currentUser + "/" + this.sifra + "/" + objectName;
          const storageRef = ref(storage, imageName);

        await uploadBytes(storageRef, blobData);
          console.log("Image " + imageName + " uploaded!");
          this.currentPictureObj.uploaded = true;
          this.currentPictureObj.name = objectName;
          this.mode = false;
          this.isUploading = false;
          pictures.mode = false;

        let url = await getDownloadURL(ref(storage, imageName))
          this.currentPictureObj.url = url;
          document.getElementById('scroll-slike').scrollIntoView(true);
      }
      catch (e) {
        console.error(e);
      }
    },
    //Odaberi sliku za spremiti
    async imageUpload(image) {
      this.refresh = false;
      document.getElementById('scroll-slike').scrollIntoView(true);
      pictures.mode = true;
      this.mode = true;
      var picture = pictures.guitarPictures.find((picture) => picture.id === image);
      this.currentPictureObj = picture;
      this.currentPictureObj.url = '';
      this.uploadText = picture.text;

      await wait(0.275);
        this.isUploadingPicture[image] = true;
        this.refresh = true;
    },
    async otkupi() {
      try {
        await setDoc(doc(db, "zahtjevi", this.sifra), {
          korisnik: store.currentUser,
          instrument: [
            this.odabranaVrsta,
            this.odabraniProizvodac,
            this.odabraniModel,
            this.odabranaSerija,
            this.godinaProizvodnje,
            this.vlasnik,
            this.stanje,
          ],
          slike:[
            pictures.guitarPictures[0].url,
            pictures.guitarPictures[1].url,
            pictures.guitarPictures[2].url,
            pictures.guitarPictures[3].url,
            pictures.guitarPictures[4].url,
            pictures.guitarPictures[5].url,
          ],
          zahtjevPredan: Date.now(),
          napomena: this.napomena,
          status: "U razradi",
          preporucenaCijena: Math.round(this.preporucenaCijena),
        });
        console.log("Predan zahtjev za otkup");
        this.zahtjevPredan = true;
        this.sendEmail();
        router.push('status-otkupa');
      } catch (e) {
        console.error("Greška kod predaje zahtjeva: ", e);
      }
    },
    async sendEmail(){
      this.fetchUserData;
      var params = {
        ime: this.imePrezime,
        email: this.email,
        oib: this.oib,
        mob: this.mob,
        vrsta: this.odabranaVrsta,
        proizvodac: this.odabraniProizvodac,
        model : this.odabraniModel,
        serija: this.odabranaSerija,
        godina_proizvodnje: this.godinaProizvodnje,
        vlasnik: this.vlasnik,
        stanje: this.stanje,
        cijena: this.preporucenaCijena,
        sifra: this.sifra,
        napomena: this.napomena,
        iban: this.iban,
      };
      let emailSent = await emailjs.send("service_ox0wdn1", "noviZahtjev", params);
        console.log("Email sent! ", emailSent.text);
    },
  },
  computed: {
    stanjaSet() {
      if (this.vlasnik == "1") 
        this.prikazanaStanja =  ['Novo', 'Rabljeno', 'Neispravno'];
      else {
        if (this.stanje != 'Neispravno') 
          this.stanje = 'Rabljeno';
        this.prikazanaStanja =  ['Rabljeno', 'Neispravno'];
      }
      return this.prikazanaStanja;
    },
    izracunCijene() {
      for (var element in this.cijeneSerija)
        if (this.odabranaSerija === this.cijeneSerija[element].serija)
          this.preporucenaCijena = this.cijeneSerija[element].cijena;
      var cijenaInstrumenta = this.preporucenaCijena;

      switch (parseInt(this.vlasnik)) {
        case 1:
          this.preporucenaCijena = cijenaInstrumenta;
          break;
        case 2:
          this.preporucenaCijena -= 0.1 * this.preporucenaCijena;
          break;
        case 3:
          this.preporucenaCijena -= 0.17 * this.preporucenaCijena;
          break;
      }

      switch (this.stanje) {
        case "Novo":
          this.preporucenaCijena += this.preporucenaCijena * 0.1;
          break;
        case "Rabljeno":
          this.preporucenaCijena -= this.preporucenaCijena * 0.30;
          break;
        case "Neispravno":
          this.preporucenaCijena -= this.preporucenaCijena * 0.55;
          break;
      }

      var year = parseInt(this.godinaProizvodnje);
      if (year >= 1950 && year < 1960)
        this.preporucenaCijena += this.preporucenaCijena * 2.00;
      else if (year >= 1960 && year < 1970)
        this.preporucenaCijena += this.preporucenaCijena * 1.5;
      else if (year >= 1970 && year < 1980)
        this.preporucenaCijena += this.preporucenaCijena * 0.5;
      else if (year >= 1980 && year < 2000)
        this.preporucenaCijena += this.preporucenaCijena * 0.2;
      else if (year >= 2000 && year < 2010)
        this.preporucenaCijena += this.preporucenaCijena * 0.1;
      this.preporucenaCijena;

      if (this.yearCheck || !this.godinaProizvodnje) return 0;
      else if (this.preporucenaCijena <= 0) return "ne vrijedi";
      else return Math.round(this.preporucenaCijena) + " kn";
    },
    proizvodacSet() {
      this.prikazaniProizvodaci = [];
      this.prikazaniModeli = [];
      this.prikazaneSerije = [];
      this.cijeneSerija = [];

      for (const data in this.jsonData) {
        for (const data1 in this.jsonData[data])
          this.prikazaniProizvodaci.push(`${data1}`);
        }
      this.odabraniProizvodac = this.prikazaniProizvodaci[0];
      return this.prikazaniProizvodaci;
    },
    modelSet() {
      this.prikazaniModeli = [];
      this.prikazaneSerije = [];
      this.cijeneSerija = [];
      for (const data in this.jsonData) {
        for (const data1 in this.jsonData[data]) {
          if (data1 === this.odabraniProizvodac)
            for (const data2 in this.jsonData[data][data1]) 
              this.prikazaniModeli.push(`${data2}`);
        }
      }
      this.odabraniModel = this.prikazaniModeli[0];
      return this.prikazaniModeli;
    },
    serijaSet() {
      this.prikazaneSerije = [];
      for (const data in this.jsonData) {
        for (const data1 in this.jsonData[data]) {
          for (const data2 in this.jsonData[data][data1]) {
            if (data2 === this.odabraniModel)
              for (const data3 in this.jsonData[data][data1][data2]) {
                var element = {};
                element.cijena = this.jsonData[data][data1][data2][data3];
                element.serija = `${data3}`;
                this.prikazaneSerije.push(element.serija);
                this.cijeneSerija.push(element);
              }
          }
        }
      }
      this.odabranaSerija = this.prikazaneSerije[0];
      return this.prikazaneSerije;
    },
    picturesCheck() {
      return pictures.guitarPictures.every(picture => picture.uploaded);
    },
    allFilled() {
      return this.odabranaVrsta &&
        this.odabraniProizvodac &&
        this.odabraniModel &&
        this.odabranaSerija &&
        !this.yearCheck &&
        this.godinaProizvodnje &&
        this.vlasnik &&
        this.stanje &&
        this.preporucenaCijena > 0 &&
        this.preporucenaCijena !== "ne vrijedi" &&
        !this.stanjeCheck &&
        this.picturesCheck
        ? 1
        : 0;
    },
    yearCheck() {
      return (this.godinaProizvodnje < 1950 ||
        this.godinaProizvodnje > new Date().getFullYear()) &&
        this.godinaProizvodnje
        ? 1
        : 0;
    },
  },
};
</script>

<style scoped lang="scss">
.otkup-textarea:focus {
  outline: none !important;
  border-color: var(--FluorescentRed__FrenchWine);
}
.menu-bottom {
  position: fixed;
  bottom: -1px;
  width: 100%;
  height: 50px;
  background-color: var(--BalticSea__DarkToneInk);
  box-shadow: 0px -4px 4px var(--Transparent25__Transparent75);
  .bg-bottom {
    position: absolute;
    width: 100%;
    height: 50px;
  }
  .menu-item-bottom {
    display: flex;
    align-items: center;
    font-size: 4.5vw;
    & p {
      color: var(--Snow__Lead) !important;
    }
    .money {
      width: 43px;
      opacity: 0.75;
    }
    .price {
      color: var(--FluorescentRed__FrenchWine);
    }
  }
}
.square {
  display: flex;
  background-color: var(--DustySky__Grey);
  overflow: hidden;
  float:left;
  position: relative;
  width: 100%;
  padding-bottom: 100%;
  border-color: var(--StretchLimo__ChromaphobicBlack);
  box-shadow: 0px 0px 4px var(--Transparent25__Transparent75);
  .square-img {
    position: absolute;
    background-color: var(--DustySky__Grey);
    width: 100%;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
    pointer-events: none;
  }
  .square-img2 {
    position: absolute;
    width: 100%;
    height: 100% !important;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
    color: var(--BalticSea__Squant);
    font-size: 3.2vw; 
    letter-spacing: 0.25px;
  }
  .hide-text {
    opacity: 0;
  }
  .show-text {
    opacity: 1;
  }
}

.CDodavanje-top {
  position: absolute;
  display: flex;
  align-items: center;
  text-align: center;
  background-color: var(--StretchLimo__EerieBlack);
  width: 100%;
  height: 72%;
  box-shadow: 0px 4px 4px var(--Transparent25__Transparent75);
  & div:nth-child(1) {
    align-items: center;
    text-align: center;
    & p:nth-child(2) {
      color: var(--Fresco__KinglyCloud) !important;
      letter-spacing: 0.5px;
      margin-top: 8px;
      font-size: 14px;
    }
    & p:nth-child(3) {
      color: var(--Fresco__KinglyCloud) !important;
      margin-top: 4px;
      letter-spacing: -1.5px;
      font-size: 24px;
      font-weight: bold;
    }
    & p:nth-child(4) {
      color: var(--Fresco__KinglyCloud) !important;
      margin-top: 4px;
      letter-spacing: 0px;
      font-size: 22px;
    }
    .dodavanje-icon {
      width: 100%;
      height: 123px;
    }
  }
  .loading2 {
    width: 100%;
    align-items: center;
    text-align: center;
    justify-content: center;
    color: var(--Snow__Lead);
  }
}
.CDodavanje-bottom {
  position: absolute;
  align-items: center;
  text-align: center;
  background-color: var(--Snow__DarkToneInk);
  bottom: 0px;
  width: 100%;
  height: 28%;
  .arrow {
    position: absolute;
    bottom: 0px;
    left: 0px;
    margin: 8px;
    width: 42px;
    transform: scaleX(-1);
  }
}

.croppa-container {
  width: 100% !important;
  height: 100% !important;
  position: absolute;
  color: transparent;
  height: 100% !important;
  background-color: transparent;
}

.loader-squre-img {
    position: absolute;
    display: flex;
    justify-content: center;
    width: 100%;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
}

.loading {
  align-items: center;
  text-align: center;
  justify-content: center;
  color: var(--BalticSea__BlackMana);
}

.hide-loading {
  overflow: hidden;
  height: 0px;
}

.show-loading {
  overflow: visible;
  height: auto;
}

.loader {
  border-color: var(--BalticSea__SilverMedal);
  border-top-color: var(--FluorescentRed__FrenchWine);
}

.show-ucitaj {
  position: fixed;
  top: 0px;
  right: 0px;
  width: 100%;
  height: 100%;
  z-index: 999999;
}

.hide-ucitaj {
  position: fixed;
  top: 0px;
  right: 200%;
  width: 100%;
  height: 100%;
  z-index: 999999;
}

.menu-bottom-dodavanje {
  position: absolute;
  bottom: 0px;
  width: 100%;
  height: 50px;
  background-color: transparent;
  & div {
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    width: 54px;
    height: 54px;
    margin-top: 23px;
    border-radius: 100%;
    background-color: var(--FluorescentRed__FrenchWine);
    z-index: 42069;
  }
}
.photo-icon {
  height: 34px;
}
.photo-iconR {
  height: 34px;
  animation:spin 1s linear infinite;
  @keyframes spin { 
    100% { transform: rotate(360deg); } 
  }
}
.photo-iconD {
  position: fixed;
  width: 0px;
  opacity: 0%;
  bottom: 0px;
  height: 0px;
}
</style>
