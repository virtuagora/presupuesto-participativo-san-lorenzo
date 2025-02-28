<template>
  <section>
    <img src="/assets/img/pp-logo-horizontal.svg" class="image logo-small-pp-voting is-pulled-right" alt="">
    <h1 class="title is-3">1. Proyectos comunitarios</h1>
    <h1 class="subtitle is-5">Podés elegir hasta 3 proyectos.</h1>
    <hr>
    <div class="notification is-comunitario has-text-centered">
              <h1 class="title is-3 has-text-white">
            <img src="/assets/img/icons/w-comunitario.svg" alt="" class="image is-inline" style="width: 35px;margin-bottom: -8px;">
               Proyectos comunitarios
              </h1>
            </div>
    <div class="columns">
      <div class="column">
        <div class="field">
          <div class="control">
            <div class="select is-medium is-fullwidth">
              <select v-model.number="selectedDistrict" placeholder="Seleccione el tipo de doc">
                <option :value="null">- Todos los distritos -</option>
                <option :value="1">Distrito NORTE</option>
                <option :value="2">Distrito CENTRO</option>
                <option :value="3">Distrito SUR</option>
              </select>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="card is-checkbox voting-box" :class="colorBackground(proyecto.id)" v-for="proyecto in visibleProjects" :key="`voting-box-${proyecto.id}`" @click="toggleCheckbox(proyecto)">
      <div class="card-content">
        <article class="media">
          <div class="media-left">
            <h1 class="title is-1 is-marginless is-checkbox" :class="colorChecked(proyecto.id)">
              <span v-if="checked.includes(proyecto.id)">✔</span>
              <span v-else>☐</span>
            </h1>
          </div>
          <div class="media-content">
            <h1 class="title is-4 is-marginless">{{proyecto.name}}</h1>
            <p class="some-good-effects">Por <span class="is-600">{{getWho(proyecto)}}</span> - {{getShortDescription(proyecto.objective,150)}}</p>
          </div>
          <div class="media-right">
            <div class="control">
              <div class="tags has-addons">
                <span class="tag is-dark">
                  <i class="fas fa-dollar-sign"></i>
                </span>
                <span class="tag is-warning" style="flex:1">{{parseInt(proyecto.total_budget.replace('.00','')).toLocaleString('es')}}</span>
              </div>
            </div>
            <div class="control">
              <span class="tag is-primary" style="width: 100%;">Distrito {{getDistrict(proyecto.district_id)}}</span>
            </div>
            <div class="control">
              <span class="tag is-light" style="width: 100%;">{{proyecto.code}}</span>
            </div>
          </div>
        </article>
      </div>
    </div>
    <hr>
    <h1 class="title is-5">Estos son los proyectos que elegiste</h1>
    <h1 class="subtitle is-6">(Podés cambiarlo antes de enviar tu voto)</h1>
    <div class="notification is-comunitario" v-for="voteProyecto in voteComunitarios" :key="voteProyecto.id">
      <button class="delete" @click="toggleCheckbox(voteProyecto)"></button>
      <h1 class="title is-4 is-marginless has-text-white">{{voteProyecto.name}}</h1>
      <p class="some-good-effects has-text-white">Por <span class="is-600">{{getWho(voteProyecto)}}</span> - {{getShortDescription(voteProyecto.objective,150)}}</p>
    </div>
    <b-message type="is-success" v-if="voteComunitarios.length == 0">
        <div class="has-text-centered">
        No seleccionaste proyectos comunitarios.<br><span class="is-size-7">Se considera tu voto como voto en blanco en esta categoria.</span>
        </div>
      </b-message>
    <div class="is-clearfix">
      <button v-on:click.prevent="$router.push({ name: 'PublicStep2'})" class="button is-primary is-rounded is-medium is-pulled-right">
        Continuar&nbsp;
        <i class="fas fa-arrow-right fa-fw "></i>
      </button>
    </div>
  </section>
</template>

<script>
import { indexOf, without } from "lodash";

export default {
  data() {
    return {
      checked: [],
      selectedDistrict: null,
      visibleProjects: []
    };
  },
  beforeMount: function() {
    this.checked = this.voteComunitarios.map(pro => {
      return pro.id;
    });
    this.visibleProjects = this.$store.state.comunitarios;
  },
  methods: {
     getWho(project){
      if(project.type == 'institucional'){
        return project.organization_name
      }
        return project.author_names + ' ' + project.author_surnames
    },
    isChecked: function(id) {
      return {
        "far fa-square": this.checked.indexOf(id) == -1,
        "fas fa-check-square": this.checked.indexOf(id) >= 0
      };
    },
    colorChecked: function(id) {
      return {
        "has-text-comunitario": this.checked.indexOf(id) >= 0
      };
    },
    colorBackground: function(id) {
      return {
        "is-comunitario": this.checked.indexOf(id) >= 0
      };
    },
    toggleCheckbox: function(proyecto) {
      const index = indexOf(this.checked, proyecto.id);
      if (index !== -1) {
        this.checked = without(this.checked, proyecto.id);
        this.$store.commit("removeComunitario", proyecto);
        return;
      }
      if (this.checked.length === 3) {
        this.$buefy.snackbar.open({
          message: "¡Recuerde! No puede votar mas de 3 proyectos",
          type: "is-warning",
          actionText: "OK"
        });
        return;
      }
      this.checked = [...this.checked, proyecto.id];
      this.$store.commit("addComunitario", proyecto);
      return;
    },
    getDistrict: function(id) {
      switch (id) {
        case 1:
          return "Norte";
        case 2:
          return "Centro";
        case 3:
          return "Sur";
      }
    },
    getShortDescription: function(text, limit) {
      if (text.length > limit) {
        for (let i = limit; i > 0; i--) {
          if (
            text.charAt(i) === " " &&
            (text.charAt(i - 1) != "," ||
              text.charAt(i - 1) != "." ||
              text.charAt(i - 1) != ";")
          ) {
            return text.substring(0, i) + "...";
          }
        }
      } else {
        return text;
      }
    },
  },
  computed: {
    comunitarios: function() {
      return this.$store.state.comunitarios;
    },
    voteComunitarios: function() {
      return this.$store.state.voteComunitarios;
    }
  },
  watch:{
    selectedDistrict: function(newVal, oldVal){
      if(newVal == null){
        this.visibleProjects = this.comunitarios;
      } else { 
        this.visibleProjects = this.comunitarios.filter(project => {
          return project.district_id == newVal
        })
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.media-right .control {
  margin: 1px 0;
}
.is-checkbox {
  cursor: pointer;
}
.voting-box.is-comunitario{
  color: #FFF !important;
  .title{
    color: #FFF !important;
  }
}
.some-good-effects{
  line-height: normal;
  font-size:0.85rem;
}
</style>