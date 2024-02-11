<!-- <template>
  <img alt="Vue logo" src="./assets/logo.png">
  <h1>Welcome to Your Vue.js App</h1>
</template> -->

<template>
  <v-app id="inspire">
    <div v-if="showMenu">
      <v-navigation-drawer
       fixed
       :clipped="$vuetify.breakpoint.mdAndUp"
       app
       v-model="drawer"
       width="220"
      >
        <mac-aside-menu :userName="user.usr_nome" ></mac-aside-menu>
      </v-navigation-drawer>
      <v-toolbar :clipped-left="$vuetify.breakpoint.lgAndUp" app fixed color="blue darken-2">
        <v-toolbar-title  color="grey darken-3" class="ml-0 pl-3 width300px">
          <v-toolbar-side-icon @click.stop="drawer = !drawer" class="white--text">
          </v-toolbar-side-icon>
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <v-menu
          v-model="menu"
          :close-on-content-click="false"
          :nudge-width="200"
          offset-x
        >
          <template v-slot:activator="{ on }">
            <v-btn dark icon class="white--text" v-on="on">
                <v-avatar size="32px" tile>
                  <v-icon>apps</v-icon>
                </v-avatar>
              </v-btn>
          </template>
           <v-card max-width="150" min-width="200">
            <v-list  expand three-line>
              <v-list-tile avatar>
                <v-list-tile-avatar>
                  <img
                  style="background-color:blue"
                  round
                  src="https://img2.gratispng.com/20180506/afw/kisspng-logo-technology-digital-restaurant-marketing-for-5aef5d8a830073.6054585215256364905366.jpg" alt="John">
                </v-list-tile-avatar>
                <v-list-tile-content>
                  <v-list-tile-title>
                    <span style="font-size:12px">
                      {{company.emp_descricao}}
                    </span>
                  </v-list-tile-title>
                </v-list-tile-content>
              </v-list-tile>
            </v-list>
            <v-divider></v-divider>
            <v-list>
              <router-link
              tag="v-list-tile"
              :to="{name:'profile.user'}"
              class="submenu hidden-sm-and-down pointer"
              active-class="blue--text"
              >
                <v-list-tile-action style="padding:0!important;margin:0!important">
                  <v-icon style="font-size:18px;">person</v-icon>
                </v-list-tile-action>
                <v-list-tile-title><span style="font-size:12px">Perfil</span></v-list-tile-title>
              </router-link>
              <router-link
              tag="v-list-tile"
              :to="{name:'profile.user'}"
              class="submenu hidden-sm-and-down pointer"
              active-class="blue--text"
              >
                <v-list-tile-action>
                  <v-icon  style="font-size:18px;">build</v-icon>
                </v-list-tile-action>
                <v-list-tile-title>
                  <span style="font-size:12px">Configurações</span>
                </v-list-tile-title>
              </router-link>

            </v-list>
            <div></div>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn flat @click="logout">Sair</v-btn>
            </v-card-actions>
          </v-card>
        </v-menu>
      </v-toolbar>
    </div>
    <v-content>
      <v-container fluid style="scroll"  grid-list-xs>
        <transition name="slide" mode="out-in" appear>
          <router-view ></router-view>
        </transition>
      </v-container>
    </v-content>
    <v-snackbar
     v-model="snakeBar.snackbar"
     :top="snakeBar.y === 'top'"
     :multi-line="snakeBar.mode"
     :right="snakeBar.x === 'right'"
     :timeout="snakeBar.timeout"
     :color="snakeBar.color"
     :vertical="snakeBar.mode === 'vertical'"
     >
      {{ snakeBar.text }}
      <v-btn color="white" flat @click="snakeBar.snackbar = false">
        x
      </v-btn>
    </v-snackbar>
  </v-app>
</template>

<script>
/* eslint-disable camelcase */
import { mapState } from 'vuex';
import socketIO from 'socket.io-client';
import AsideMenu from './components/Content/AsideMenu';
import { AUTH_TOKEN } from './plugins/apollo';
import AuthService from './features/Auth/services/auth-service';
// foi marcado por min import { getCompany } from './features/helpers/sessionStorageItens';
// import UserService from './features/Auth/services/user-service';

export default {
  name: 'AppComponent',
  data() {
    return {
      drawer: true,
      headerMenu: [
        { title: 'Perfil', action: 'profile' },
        { title: 'Sair', action: 'logout' },
      ],
      mini: true,
      company: {
        id: '',
        emp_descricao: '',
        emp_cnpj: '',
      },
      fav: true,
      menu: false,
      message: false,
      hints: true,
      userName: '',
    };
  },
  props: { source: String },
  computed: {
    ...mapState('Login', [
      'snakeBar',
      'showMenu',
      'companyData',
      'user',
    ]),
  },
  watch: {
    showMenu() {
      if (!this.showMenu) this.$store.dispatch('Login/showMenu', false).then();
    },
    companyData() {
      if (this.companyData) {
        const { emp_descricao } = this.companyData;
        const desc = emp_descricao.substr(0, 15);
        this.companyData.emp_descricao = `${desc}...`;
        this.company = this.companyData;
      }
    },
  },
  components: { macAsideMenu: AsideMenu },
  methods: {
    logout() {
      this.$store.dispatch('Login/logout').then((res) => {
        if (res) {
          this.$store.dispatch('Login/showMenu', false).then();
          this.$store.dispatch('Dashboard/setMenuFilteredStatus', { status: false });
          this.$router.push({ name: 'login' });
        }
      });
    },
  },
  async beforeMount() {
    const token = sessionStorage.getItem(AUTH_TOKEN);
    if (!token) {
      this.$store.dispatch('Login/showMenu', false);
      this.$store.dispatch('Dashboard/setMenuFilteredStatus', { status: false });
    } else {
      const userData = await AuthService.user({ fetchPolicy: 'network-only' });
      if (userData) {
        this.company = userData.empresa;
        this.userName = userData.usr_nome;
        this.$store.dispatch('Login/setUser', userData);
        this.$store.dispatch('Dashboard/mountMenu', { userData, status: true });
        this.$store.dispatch('Login/showMenu', true).then();
        this.$router.push({ name: 'home.dashboard' });
      }
    }
  },
  mounted() {
    const io = socketIO('http://localhost:3000');
    io.on('connect', () => {
      console.log('connect')
    })
  },
};
</script>
<style>

/* WIDTH */
.width300px{width: 300px!important;}
.width30per{width: 30%!important;}
.width49per{width: 49%!important;}
.width70per{width: 70%!important;}
.width100per{width: 100%!important;}

/* FONTS and TEXT ALG */
.fontSz07em{font-size:0.7em!important;}
.textAlgRgt{text-align: right!important;}

/* MARIGNS AND PAGGINDGS */
.marginT20{margin-top:-20px!important;}
.marginB0{margin-bottom:0!important;}
.marginT0{margin-top:0!important;}
.marginT20B0{margin-top:-20px;margin-bottom:0!important}
.marginLR20{margin:0 20px!important;}

.paddingA10{ padding:10px!important;}
.paddingR5{ padding-right: 5px!important;}
.paddingR10{ padding-right:10px!important;}

.paddingB10{ padding-bottom:15px!important;}
/* FLOATS */
.floatLft{float:left!important}

/* COLORS BG and FONT */
.bgColorFFF{background-color:#ffffff!important;}
.colorFFF{background-color:#ffffff!important;}

/* HELPERS */
.v-card-title{background-color: #1E88E5;color:#fff;max-height: 45px!important;}
.pointer { cursor: pointer !important;}
.noTextDecoration{text-decoration: none!important;}
.codMun{ width: 30%;float:left!important;padding-right:5px}
.municipio{width: 70%;}
.codPais{ width: 30%;float:left!important;padding-right:2px}
.pais{width: 70%;}
.scroll{height: 590px!important;overflow-y: scroll!important;}
.customerEmail{ width:50%!important;padding-right:20px!important;}
.customerObs{width:50%!important;float:right!important;}

/* UL ERRORS */
.listError {
  list-style-type:none!important;
  padding-left:0!important;
}
/* TRANSITIONS */

.slide-enter-active {
    animation: slide-in 200ms ease-out forwards;
}
.slide-leave-active {
    animation: slide-out 200ms ease-out forwards;
}

@keyframes slide-in {
    from {
        transform: translateX(-50px);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}
@keyframes slide-out {
    from {
        transform: translateX(0);
        opacity: 1;
    }
    to {
        transform: translateX(-50px);
        opacity: 0;
    }
}

/* MEDIA */
@media (max-width: 768px){
    .statusPrd{width: 45%!important;float: left!important;margin-right: 20px!important;}

}
@media (max-width: 425px ) {
    .container{
        padding:20px 0 0 0!important;
    }
    .v-layout-mob{
        padding:0 5px!important;
    }
    .btnMob{margin: 10px!important;}
    .statusPrd{
        width: 100%!important;
        float: none!important;
        margin-right: inherit!important;
    }
    .width49per{width: 100%!important;}
    .codMun, .municipio{ width: 100%;}
    .customerEmail{ width:100%!important;}
    .customerObs{width:100%!important;padding-top:20px!important;}
}


</style>
