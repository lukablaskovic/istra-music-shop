<template>
  <div id="app">
    <div v-if="store.theme ? loadTheme() : loadTheme()"></div>
    <div class="CBg" :class="store.darkToggle ? 'CBg-light' : 'CBg-dark'"></div>
    <div v-if="store.currentUser">
      
    <!--==============DJELATNIK============================-->
      <div v-if="currentRouteName != 'Registracija' && currentRouteName != 'Prijava' && currentRouteName != 'PasswordReset'">
        <div v-if="store.currentUser == 'istramusicshop@gmail.com'" class="menu2 top-0 grid grid-cols-2 mb-4"> <!--Paragraph routes-->
      
          <div :class=" currentRouteName == 'Djelatnik' ? 'menu-item2 menu-top-item-grid0' : 
                        currentRouteName == 'Racun' ? 'menu-item2 menu-top-item-grid1' : ''"> 
          </div>
          <!--Div routes-->
          <div v-if="currentRouteName != 'PregledOtkupa'" class="menu top-0 grid grid-cols-2 mb-4">

            <div :class="currentRouteName == 'Djelatnik' ? 'menu-item-active' : 'menu-item'">
              <p><router-link to="djelatnik"> <p>Pregled otkupa</p> </router-link></p>
            </div>
            <div :class="currentRouteName == 'Racun' ? 'menu-item-active' : 'menu-item'">
              <p><router-link to="racun"> <p>Račun</p> </router-link></p>
            </div>

          </div> <!--/Div routes-->
        </div> <!--/Paragraph routes-->
        <!--==============/DJELATNIK=========================-->

        <!--==============KORISNIK =============================-->
        <div v-else class="menu2 top-0 grid grid-cols-3 mb-4"> <!--Paragraph routes-->

          <div :class=" currentRouteName == 'StatusOtkupa' ? 'menu-item2 menu-top2-item-grid0' : 
                        currentRouteName == 'OtkupOpreme'  ? 'menu-item2 menu-top2-item-grid1' : 
                        currentRouteName == 'Racun'        ? 'menu-item2 menu-top2-item-grid2' : ''">
          </div> <!--Div routes-->

          <div v-if="currentRouteName != 'PregledOtkupa'" class="menu top-0 grid grid-cols-3 mb-4">

            <div :class="currentRouteName == 'StatusOtkupa' ? 'menu-item-active' : 'menu-item'">
              <p><router-link to="status-otkupa"><p> Status otkupa </p></router-link></p>
            </div>
            <div :class="currentRouteName == 'OtkupOpreme' ? 'menu-item-active' : 'menu-item'">
              <p><router-link to="otkup-opreme"><p> Otkup opreme </p></router-link></p>
            </div>
            <div :class="currentRouteName == 'Racun' ? 'menu-item-active' : 'menu-item'">
              <p><router-link to="racun"><p> Račun </p></router-link></p>
            </div>

          </div> <!--/Div routes-->
        </div> <!--/Paragraph routes-->
        <!--==============/KORISNIK==========================-->
      </div>
    </div>

    <!--==============HELP==============-->
    <div v-if="currentRouteName == 'Home' || currentRouteName == 'Prijava'">
      <img @click="helpPopup = true" class="help" src="https://cdn0.iconfinder.com/data/icons/cosmo-symbols/40/help_1-512.png">
      
      <div :class="helpPopup ? 'darken-bg' : 'undarken-bg'"></div>
      
      <div @click="helpPopup = false" class="pl-4 pr-4" :class="helpPopup ? 'helpPopup' : 'closeHelp'">
        <ol class="p-4 list-disc">
          <h1 class="text-center mb-4 text-24px"><b>Kratke upute kako koristiti aplikaciju</b></h1>
          <li>Aplikacija služi svima koji žele prodati svoje glazbene instrumente. </li><br>
          <li>Prilikom registracije tražit ćemo vas OIB iz sigurnosnih razloga te radi generiranja kupoprodajnog ugovora. Također, morate unijeti IBAN računa gdje želite da Vam uplatimo novac.</li><br>
          <li>Nakon registracije odmah možete predati zahtjev za otkup na <b>OtkupOpreme</b> odabirom vašeg instrumenta među ponuđenima i popunjavanjem ostalih podataka koje vas budemo tražili.</li><br>
          <li>Tražit ćemo Vas i slike vašeg instrumenta. Zahtjev možemo odbiti ukoliko slike nisu dovoljno dobre kvalitete.</li><br>
          <li>Status vašeg zahtjeva možete pratiti na <b>StatusOtkupa</b>. Obratite pažnju ukoliko Vam naš djelatnik predloži drugu cijenu za vaš instrument, možete ju prihvatiti ili odbiti. </li><br>
        </ol>
      </div>
    </div>
    <!--==============HELP==============-->

    <router-view />
  </div>
</template>
<script>
import store from "@/store";
import pictures from "@/pictures";
import router from "@/router";

import { collection, getDocs } from "@/firebase";
import { db } from "@/firebase";
import { getAuth, onAuthStateChanged } from "@/firebase";

const auth = getAuth();

onAuthStateChanged(auth, (user) => {
  const currentRoute = router.currentRoute;
  if (user) {
    console.log("LOGGED IN: " + user.email);
    store.currentUser = user.email;
    store.userID = user.uid;
    store.emailVerified = user.emailVerified;

    if(!user.emailVerified && currentRoute.name != "Registracija"){
      console.log("Error, email not verified")
      router.push({ name: "Prijava" }).catch(error => {});
    }
    else {
    //Posebni slucaj kad korisnik napusta PregledOtkupa (dodano zbog gubitka podataka)
    if (currentRoute.name == "PregledOtkupa") {
      store.currentUser === "istramusicshop@gmail.com"
        ? router.replace({ name: "Djelatnik" })
        : router.replace({ name: "StatusOtkupa" });
    }
    //Ako korisnik pokusava otic na admin stranicu
    else if(currentRoute.meta.admin && store.currentUser!= "istramusicshop@gmail.com"){
      router.push({ name: "OtkupOpreme" });
    }
    //Ako djelatnik pokusa otici na korisnik stranicu
    else if(currentRoute.meta.clientOnly && store.currentUser=== "istramusicshop@gmail.com"){
      router.push({ name: "Djelatnik" });
    }
    //Ako prijavljeni korisnik pokusava otici na Home/Login/Register
    else if (!currentRoute.meta.needsUser) {
      store.currentUser === "istramusicshop@gmail.com"
        ? router.replace({ name: "Djelatnik" })
        : router.replace({ name: "OtkupOpreme" });
    }
    }
  } 
  //Nema korisnika
  else {
    console.log("NO USER");
    store.currentUser = null;
    if (currentRoute.meta.needsUser) {
      router.push({ name: "Prijava" }).catch(error => {});
    }
  }
});


export default {
  name: "app",
  data() {
    return {
      store,
      pictures,
      helpPopup: false,
    };
  },
  mounted() {
    this.setTheme();
  },
  methods: {
    async setTheme() {
      const querySnapshot = await getDocs(collection(db, "users"));
      querySnapshot.forEach((doc) => {
        if (store.currentUser === `${doc.data().email}`) {
          store.theme = `${doc.data().theme}`;
          store.userID = `${doc.id}`;
        }
      });
    },
    loadTheme() {
      if (store.theme == "Svijetla") {
        store.darkToggle = true;
        this.loadLight();
      }
      else if (store.theme == "Tamna Plava") {
        store.darkToggle = false;
        this.loadDarkBlue();
      }
      else if (store.theme == "Tamna Crvena") {
        store.darkToggle = false;
        this.loadDarkRed();
      }
    },
    loadDarkBlue() {
      let r = document.querySelector(':root');
        r.style.setProperty('--Snow__DarkToneInk',                          '#121212');   
        r.style.setProperty('--Lateo__DarkToneInk',                         '#121212'); 
        r.style.setProperty('--DustySky__Black',                            '#000000');  
        r.style.setProperty('--DustySky__Grey',                             '#2a2a2a');    
        r.style.setProperty('--BalticSea__Lead',                            '#ffffffde'); 
        r.style.setProperty('--BalticSea__Squant',                          '#ffffff99'); 
        r.style.setProperty('--BalticSea__BlackMana',                       '#ffffff7a'); 
        r.style.setProperty('--BalticSea__SilverMedal',                     '#ffffff29'); 
        r.style.setProperty('--Transparent__BleachedSilk',                  '#ffffff0d'); 
        r.style.setProperty('--BalticSea__DarkToneInk',                     '#303030');  
        r.style.setProperty('--BalticSea__EerieBlack',                      '#1c1c1c');  
        r.style.setProperty('--Snow__Lead',                                 '#ffffffde');  
        r.style.setProperty('--FluorescentRed__FrenchWine',                 '#10498b'); 
        r.style.setProperty('--FluorescentRed__OffBlack',                   '#303030'); 
        r.style.setProperty('--Transparent__ExtraordinaryAbundanceOfTinge', '#ffffff1a'); 
        r.style.setProperty('--VivaldiRed__FrenchWine',                     '#ad1f47'); 
        r.style.setProperty('--Flushed__LingonberryRed',                    '#2f709c');
        r.style.setProperty('--PaleFlower__RavensBanquet',                  '#1e5497'); 
        r.style.setProperty('--Tin__WhatWeDoInTheShadows',                  '#00123d'); 
        r.style.setProperty('--SolidEmpire__DarkSouls',                     '#ffffff5c'); 
        r.style.setProperty('--Fresco__KinglyCloud',                        'ffffffde'); 
        r.style.setProperty('--GraniteBrown__Lead',                         '#000000de');   
        r.style.setProperty('--EnglishBreakfast__RavensBanquet',            '#1e5497'); 
        r.style.setProperty('--LimonFresco__Anime',                         '#cec14c'); 
        r.style.setProperty('--SnowPea__RichGreen',                         '#258843'); 
        r.style.setProperty('--ForbiddenFruit__RedPotion',                  '#215b9e'); 
        r.style.setProperty('--DustySky__Lead',                             '#ffffffde'); 
        r.style.setProperty('--StretchLimo__ChromaphobicBlack',             'transparent'); 
        r.style.setProperty('--StretchLimo__EerieBlack',                    '#1c1c1c'); 
        r.style.setProperty('--DwarfFortress__Belladonna',                  '#09031f'); 
        r.style.setProperty('--DwarfFortress__Lead',                        '#ffffffde'); 
        r.style.setProperty('--White__DarkGrey',                            '#363636'); 
        r.style.setProperty('--Grey__ExistentialAngst',                     '#0a0a0a'); 
        r.style.setProperty('--FluorescentRed__RefinedChianti',             '#8c1939'); 
        r.style.setProperty('--HumorousGreen__Anime',                       '#c9bc4b'); 
        r.style.setProperty('--Transparent50__Transparent25',               '0.25');    
        r.style.setProperty('--Transparent25__Transparent75',               '#000000bf');    
    },
    loadDarkRed() {
      let r = document.querySelector(':root');
        r.style.setProperty('--Snow__DarkToneInk',                          '#121212');   
        r.style.setProperty('--Lateo__DarkToneInk',                         '#121212');  
        r.style.setProperty('--DustySky__Black',                            '#000000');  
        r.style.setProperty('--DustySky__Grey',                             '#2a2a2a');    
        r.style.setProperty('--BalticSea__Lead',                            '#ffffffde'); 
        r.style.setProperty('--BalticSea__Squant',                          '#ffffff99'); 
        r.style.setProperty('--BalticSea__BlackMana',                       '#ffffff7a'); 
        r.style.setProperty('--BalticSea__SilverMedal',                     '#ffffff29'); 
        r.style.setProperty('--Transparent__BleachedSilk',                  '#ffffff0d'); 
        r.style.setProperty('--BalticSea__DarkToneInk',                     '#303030');  
        r.style.setProperty('--BalticSea__EerieBlack',                      '#1c1c1c');  
        r.style.setProperty('--Snow__Lead',                                 '#ffffffde');  
        r.style.setProperty('--FluorescentRed__FrenchWine',                 '#ad1f47'); 
        r.style.setProperty('--FluorescentRed__OffBlack',                   '#303030'); 
        r.style.setProperty('--Transparent__ExtraordinaryAbundanceOfTinge', '#ffffff1a'); 
        r.style.setProperty('--VivaldiRed__FrenchWine',                     '#ad1f47'); 
        r.style.setProperty('--Flushed__LingonberryRed',                    '#cc4658');
        r.style.setProperty('--PaleFlower__RavensBanquet',                  '#bb2a53'); 
        r.style.setProperty('--Tin__WhatWeDoInTheShadows',                  '#400F21'); 
        r.style.setProperty('--SolidEmpire__DarkSouls',                     '#ffffff5c'); 
        r.style.setProperty('--Fresco__KinglyCloud',                        'ffffffde'); 
        r.style.setProperty('--GraniteBrown__Lead',                         '#000000de');   
        r.style.setProperty('--EnglishBreakfast__RavensBanquet',            '#bb2a53'); 
        r.style.setProperty('--LimonFresco__Anime',                         '#cec14c'); 
        r.style.setProperty('--SnowPea__RichGreen',                         '#258843'); 
        r.style.setProperty('--ForbiddenFruit__RedPotion',                  '#e41a3c'); 
        r.style.setProperty('--DustySky__Lead',                             '#ffffffde'); 
        r.style.setProperty('--StretchLimo__ChromaphobicBlack',             'transparent'); 
        r.style.setProperty('--StretchLimo__EerieBlack',                    '#1c1c1c'); 
        r.style.setProperty('--DwarfFortress__Belladonna',                  '#1f0309'); 
        r.style.setProperty('--DwarfFortress__Lead',                        '#ffffffde'); 
        r.style.setProperty('--White__DarkGrey',                            '#363636'); 
        r.style.setProperty('--Grey__ExistentialAngst',                     '#0a0a0a'); 
        r.style.setProperty('--FluorescentRed__RefinedChianti',             '#8c1939'); 
        r.style.setProperty('--HumorousGreen__Anime',                       '#c9bc4b'); 
        r.style.setProperty('--Transparent50__Transparent25',               '0.25');    
        r.style.setProperty('--Transparent25__Transparent75',               '#000000bf');    
    },
    loadLight() {
      let r = document.querySelector(':root');
        r.style.setProperty('--Snow__DarkToneInk',                          '#fff9f9'); 
        r.style.setProperty('--Lateo__DarkToneInk',                         '#e8e7e7');  
        r.style.setProperty('--DustySky__Black',                            '#e6dcdc'); 
        r.style.setProperty('--DustySky__Grey',                             '#fff9f9');  
        r.style.setProperty('--BalticSea__Lead',                            '#3d3d3f'); 
        r.style.setProperty('--BalticSea__SilverMedal',                     '#3d3d3f'); 
        r.style.setProperty('--BalticSea__Squant',                          '#3d3d3f'); 
        r.style.setProperty('--Transparent__BleachedSilk',                  '#3d3d3f00'); 
        r.style.setProperty('--BalticSea__DarkToneInk',                     '#3d3d3f'); 
        r.style.setProperty('--Snow__Lead',                                 '#fff9f9'); 
        r.style.setProperty('--BalticSea__BlackMana',                       '#3d3d3f'); 
        r.style.setProperty('--BalticSea__EerieBlack',                      '#3d3d3f'); 
        r.style.setProperty('--FluorescentRed__FrenchWine',                 '#ff5252'); 
        r.style.setProperty('--FluorescentRed__OffBlack',                   '#ff5252'); 
        r.style.setProperty('--Transparent__ExtraordinaryAbundanceOfTinge', '#e8e7e700'); 
        r.style.setProperty('--VivaldiRed__FrenchWine',                     '#e93a3a'); 
        r.style.setProperty('--Flushed__LingonberryRed',                    '#e45656'); 
        r.style.setProperty('--PaleFlower__RavensBanquet',                  '#668AAC'); 
        r.style.setProperty('--Tin__WhatWeDoInTheShadows',                  '#919192'); 
        r.style.setProperty('--SolidEmpire__DarkSouls',                     '#635a5a'); 
        r.style.setProperty('--Fresco__KinglyCloud',                        '#f5dada'); 
        r.style.setProperty('--GraniteBrown__Lead',                         '#3f2c28'); 
        r.style.setProperty('--EnglishBreakfast__RavensBanquet',            '#461717'); 
        r.style.setProperty('--LimonFresco__Anime',                         '#cec14c'); 
        r.style.setProperty('--SnowPea__RichGreen',                         '#6dd277'); 
        r.style.setProperty('--ForbiddenFruit__RedPotion',                  '#ff8080'); 
        r.style.setProperty('--DustySky__Lead',                             '#96a4a5'); 
        r.style.setProperty('--StretchLimo__ChromaphobicBlack',             'transparent'); 
        r.style.setProperty('--StretchLimo__EerieBlack',                    '#292e32'); 
        r.style.setProperty('--DwarfFortress__Belladonna',                  '#1a0505'); 
        r.style.setProperty('--DwarfFortress__Lead',                        '#1a0505'); 
        r.style.setProperty('--White__DarkGrey',                            '#ffffff'); 
        r.style.setProperty('--Grey__ExistentialAngst',                     '#808080'); 
        r.style.setProperty('--FluorescentRed__RefinedChianti',             '#ff5252'); 
        r.style.setProperty('--HumorousGreen__Anime',                       '#c9ba39'); 
        r.style.setProperty('--Transparent50__Transparent25',               '0.50');  
        r.style.setProperty('--Transparent25__Transparent75',               '#00000040');    
    },
  },
  computed: {
    currentRouteName() {
      return this.$route.name;
    },
  },
};
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Alike&display=swap");

.CBg {
  position: fixed;
  left: 0;
  right: 0;
  z-index: -9999;
  display: block;
  background-size: 50%;
  background-repeat: repeat;
  width: 100%;
  height: 200%;
  bottom: -50px;
}
.CBg-light {
  background-image: url("assets/bg_light.png");
}
.CBg-dark {
  background-image: url("assets/bg_dark.png");
}
.menu2 {
  position: fixed;
  top: 0px;
  width: 100%;
  height: 50px;
  z-index: 2000;
  background-color: var(--BalticSea__DarkToneInk);
  box-shadow: 0px 4px 4px var(--Transparent25__Transparent75);
  .menu-item2 {
    position: absolute;
    height: inherit;
    width: inherit;
    transition: all;
    transition-timing-function: ease-in-out;
    transition: 0.5s;
    grid-column-start: 1;
    grid-column-end: 2;
    display: flex;
    align-items: center;
    text-align: center;
    z-index: 100;
    font-size: 4.5vw;
  }
}
.menu-top-item-grid0 {
  margin-left: 0%;
  background-color: var(--FluorescentRed__FrenchWine);
}
.menu-top-item-grid1 {
  margin-left: 100%;
  background-color: var(--FluorescentRed__FrenchWine);
}
.menu-top2-item-grid0 {
  margin-left: 0%;
  background-color: var(--FluorescentRed__FrenchWine);
}
.menu-top2-item-grid1 {
  margin-left: 100%;
  background-color: var(--FluorescentRed__FrenchWine);
}
.menu-top2-item-grid2 {
  margin-left: 200%;
  background-color: var(--FluorescentRed__FrenchWine);
}

.menu-item-active {
  display: flex;
  align-items: center;
  text-align: center;
  transition: all;
  transition-timing-function: ease-in-out;
  transition: 0.25s;
  background-color: var(--Transparent__BleachedSilk);
  font-size: 4.5vw;
  & p {
    width: 100%;
    font-weight: bold;
    color: var(--graniteBrown);
    transition: all;
    transition-timing-function: ease-in-out;
    transition: 0.1s !important;
    font-size: 4.5vw;
    & a {
      width: 100%;
      font-weight: bold;
      color: var(--graniteBrown);
      transition: all;
      transition: 0.25s;
      transition-timing-function: ease-in-out;
      font-size: 4.5vw;
    }
  }
}

.menu {
  position: fixed;
  top: 0px;
  width: 100%;
  height: 50px;
  z-index: 3000;
  background-color: transparent;
}

.menu-item {
  display: flex;
  align-items: center;
  text-align: center;
  background-color: var(--Transparent__BleachedSilk);
  font-size: 4.5vw;
  & p {
    width: 100%;
    height: 100%;
    color: var(--Fresco__KinglyCloud);  
    transition: all;
    transition-timing-function: ease-in-out;
    font-size: 4.5vw;
    & a {
      width: 100%;
      color: var(--Fresco__KinglyCloud);
      transition: all;
      transition-timing-function: ease-in-out;
      font-size: 4.5vw;
      & p {
        display: flex;
        align-items: center;
        text-align: center;
        justify-content: center;
        height: 100%;
      }
    }
    & b {
      width: 100%;
      font-size: 4.5vw;
    }
  }
}
.help {
  position: absolute;
  right: 0px;
  bottom: 0px;
  height: 40px;
  margin: 4px;
}
.undarken-bg {
  top: 0px;
  width: 100%;
  height: 100%;
  position: fixed;
  z-index: -100;
  background-color: hsl(0, 0%, 0%, 0%);
  backdrop-filter: blur(0px);
}
.darken-bg {
  top: 0px;
  width: 100%;
  height: 100%;
  position: fixed;
  z-index: 40000000;
  background-color: hsl(0, 0%, 0%, 50%);
  backdrop-filter: blur(4px);
}

.helpPopup {
  position: absolute;
  left:50%;
  top:50%;
  transform:translate(-50%,-50%);
  position: fixed !important;
  z-index: 50000000;
  width: 90% !important;
  height: 400px;
  overflow-y: scroll;
  border-radius: 16px;
  background-color: var(--Snow__DarkToneInk);
  box-shadow: 0px 0px 8px hsl(0, 0%, 0%, 75%);
}
.closeHelp {
  position: absolute;
  overflow: hidden;
  left:50%;
  top:50%;
  transform:translate(-50%,-50%);
  width: 90% !important;
  height: 0%;
}
</style>