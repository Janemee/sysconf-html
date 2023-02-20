<template>
  <q-layout view="hHh lpR fFf">

    <q-header elevated class="bg-primary text-white " height-hint="98">
      <q-toolbar>
        <q-btn dense flat round icon="menu" @click="drawer = !drawer" />

        <q-toolbar-title>
          <q-avatar>
            <!--<img src="https://cdn.quasar.dev/logo/svg/quasar-logo.svg">-->
          </q-avatar>
          Title Demo
        </q-toolbar-title>
        <!--<q-btn dense flat round icon="menu" @click="right = !right" />-->
        <!--<q-input standout="bg-teal text-white" v-model="text" label="Custom standout" />-->
      </q-toolbar>
      <q-ajax-bar
        ref="bar"
        position="bottom"
        color="accent"
        size="10px"
        skip-hijack
      />
      <q-tabs align="center">
        <q-tab @click="trigger(loadMenu(item))"  :label="item" v-for="item in tabList" :key="item" />
      </q-tabs>
    </q-header>

    <q-drawer
      v-model="drawer"
      show-if-above
      :width="200"
      :breakpoint="500"
      bordered
      content-class="bg-grey-3"
    >
      <q-scroll-area class="fit">
        <q-list v-for="(menuItem, index) in menuList" :key="index">

          <q-item clickable @click="loadMenuData(menuItem.label)" :active="menuItem.label === group" v-ripple>
            <q-item-section avatar>
              <q-icon :name="menuItem.icon" />
            </q-item-section>
            <q-item-section>
              {{ menuItem.label }}
            </q-item-section>
          </q-item>

          <q-separator v-if="menuItem.separator" />

        </q-list>
      </q-scroll-area>
    </q-drawer>

    <q-page-container>
      <!--<router-view />-->
      <Index v-bind:group="group" :app="app" ></Index>
    </q-page-container>

    <!--<q-footer elevated class="bg-grey-8 text-white">-->
    <!--<q-toolbar>-->
    <!--<q-toolbar-title>-->
    <!--<q-avatar>-->
    <!--<img src="https://cdn.quasar.dev/logo/svg/quasar-logo.svg">-->
    <!--</q-avatar>-->
    <!--Title-->
    <!--</q-toolbar-title>-->
    <!--</q-toolbar>-->
    <!--</q-footer>-->
    <!--<tableData :app="app" :group="group"></tableData>-->
  </q-layout>
</template>
import Vue from 'vue'
import axios from 'axios'
Vue.prototype.$axios = axios

<script>
  const baseUrl = 'http://localhost:8082'

  import Index from '../pages/Index'
  export default {
    components: {
      Index
    },
    data () {
      return {
        drawer: false,
        menuList: [],
        menuListMap: {},
        tabList: [],
        app: '',
        group: ''
      }
    },
    methods: {
      trigger (callback) {
        const bar = this.$refs.bar
        bar.start()
        if (typeof callback === 'function') {
          callback()
        }
        if (this.$refs.bar) {
          this.$refs.bar.stop()
        }
      },
      loadMenuData (menu) {
        this.group = menu
      },
      loadMenu (item) {
        this.app = item
        this.menuList = this.menuListMap[item]
        this.trigger(this.$axios.get(baseUrl + '/api/menu', {
          params: {
            app: this.app
          }
        }).then((response) => {
          console.log(response)
          this.menuList = response.data
        })
          .catch(() => {}))
        if (this.menuList !== undefined && this.menuList.length > 0) {
          this.loadMenuData(this.menuList[0].label)
        } else {
          this.menuList = []
          this.loadMenuData('')
        }
      }
    },
    mounted: function () {
      this.trigger(this.$axios.get(baseUrl + '/api/app')
        .then((response) => {
          this.tabList = response.data
          if (this.tabList != null) {
            this.app = this.tabList[0]
          }
        })
        .catch(() => {}))

      // if (this.tabList === null || this.tabList === undefined || this.tabList.length === 0) {
      //   this.tabList = ['page1', 'page2', 'page3']
      // }
      this.trigger(this.$axios.get(baseUrl + '/api/menuMap', {
        params: {
          app: this.app
        }
      }).then((response) => {
        console.log(response)
        this.menuListMap = response.data
      })
        .catch(() => {}))
      // this.menuListMap = {
      //     page1: [
      //       {
      //         icon: 'help',
      //         iconColor: 'primary',
      //         label: 'Help',
      //         separator: false
      //       }
      //     ],
      //     page2: [
      //       {
      //         icon: 'help',
      //         iconColor: 'primary',
      //         label: 'Help',
      //         separator: false
      //       },
      //       {
      //         icon: 'error',
      //         label: 'Spam',
      //         separator: true
      //       },
      //       {
      //         icon: 'settings',
      //         label: 'Settings',
      //         separator: false
      //       },
      //       {
      //         icon: 'feedback',
      //         label: 'Send Feedback',
      //         separator: false
      //       }
      //     ]
      // }
    }
  }
</script>
