<template>
  <div v-on:mouseenter="isCollapse = false" v-on:mouseleave="isCollapse = true" id="menuLayout">
    <el-row  >
      <el-col :span="4">
        <el-menu router default-active="2" class="grid-content el-menu-vertical-demo" @open="handleOpen" @close="handleClose" :collapse="isCollapse">
          <el-menu-item v-for="item in items" v-bind:key="item.nom" :index="item.url">
            <i :class="item.icon"></i>
            <span slot="title">{{ item.nom }}</span>
          </el-menu-item>
        </el-menu>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import FontAwesomeIcon from '@fortawesome/vue-fontawesome'
import faSearch from '@fortawesome/fontawesome-free-solid/faSearch'
export default {
  name: 'monMenu',
  data: () => ({
    items: [
      {nom: 'FormationOld', url: '/formation', icon: 'el-icon-tickets'},
      {nom: 'Planning', url: '/planning', icon: 'el-icon-date'},
      {nom: 'Formations', url: '/listeFormation', icon: 'el-icon-tickets'},
      {nom: 'ListeModule', url: '/listeModule', icon: 'el-icon-document'},
      {nom: 'Alerte', url: '/alerte', icon: 'el-icon-warning'}
    ],
    isCollapse: true
  }),
  methods: {
    log () {
      console.log('coucou')
    },
    handleOpen (key, keyPath) {
      console.log(key, keyPath)
    },
    handleClose (key, keyPath) {
      console.log(key, keyPath)
    }
  },
  computed: {
    icon () {
      return faSearch
    }
  },

  components: {
    FontAwesomeIcon
  }
}
</script>

<style>
  #menuLayout {
    margin-right: 2em;
  }
  .el-menu-vertical-demo:not(.el-menu--collapse) {
    width: 200px;
  }
  .el-menu-vertical-demo {
    height: 100vh;
  }
  #search{
    color: #37959D;
  }
  li {
    padding: 0.2em;
  }
  a {
    color: #f3f3f3;
  }
  #fieldSearch {
    height: 1em;
  }
</style>
