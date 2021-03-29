<template>
  <div>
    <b-navbar toggleable="lg" type="dark" variant="info" fixed="top">
      <b-navbar-brand href="#">
        <img width="20%" src="/Sidestream.ico" alt="logo">
      </b-navbar-brand>

      <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

      <b-collapse id="nav-collapse" is-nav>
        <b-navbar-nav>
          <b-nav-item href="#" @click="undo(lastActions)">
            Undo
            <b-icon icon="arrow-counterclockwise"></b-icon>
          </b-nav-item>
        </b-navbar-nav>

        <!-- Right aligned nav items -->
        <b-navbar-nav class="ml-auto">
          <b-nav-form>
            <b-form-input size="sm" class="mr-sm-2" placeholder="Search"></b-form-input>
            <b-button size="sm" class="my-2 my-sm-0" type="submit">Search</b-button>
          </b-nav-form>

          <b-nav-item-dropdown text="Lang" right>
            <b-dropdown-item href="#">EN</b-dropdown-item>
            <b-dropdown-item href="#">ES</b-dropdown-item>
            <b-dropdown-item href="#">RU</b-dropdown-item>
            <b-dropdown-item href="#">FA</b-dropdown-item>
          </b-nav-item-dropdown>

          <b-nav-item-dropdown right>
            <!-- Using 'button-content' slot -->
            <template #button-content>
              <em>User</em>
            </template>
            <b-dropdown-item href="#">Profile</b-dropdown-item>
            <b-dropdown-item href="#">Sign Out</b-dropdown-item>
          </b-nav-item-dropdown>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>

    Resolved:
    <b-table striped hover :items="resolved" :fields="fields">
      <template #cell(index)="data">
        {{ data.index + 1 }}
      </template>
      <template #cell(edit)="data">
        <b-button variant="danger" size="sm" @click="unresolve(data.index, resolved, unresolved, lastActions)" class="mr-1">
          unresolve
        </b-button>
      </template>
    </b-table>
    
    Unresolved:
    <b-table striped hover :items="unresolved" :fields="fields">
      <template #cell(index)="data">
        {{ data.index + 1 }}
      </template>
      <template #cell(edit)="data">
        <b-button variant="success" size="sm" @click="resolve(data.index, unresolved, resolved, lastActions)" class="mr-1">
          resolve
        </b-button>
      </template>
    </b-table>
    
    Backlog: 
    <b-table striped hover :items="backlog" :fields="fields">
      <template #cell(index)="data">
        {{ data.index + 1 }}
      </template>
      <template #cell(edit)="data">
        <b-button variant="outline-primary" size="sm" @click="unresolveBacklog(data.index, backlog, unresolved, lastActions)" class="mr-1">
          unresolve
        </b-button>
      </template>
    </b-table>
  </div>
</template>

<script>
import $ from 'jquery'
import Vue from 'vue'
import { BootstrapVue, BootstrapVueIcons } from 'bootstrap-vue'
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'

Vue.use(BootstrapVue)
Vue.use(BootstrapVueIcons)
const operatorName = 'Houcem'

export default {
  async asyncData({ $axios }) {
    try {
      const { resolved, unresolved, backlog } = await $axios.$get(
        "http://localhost:8000/get_lists", { params: { operator_name : operatorName } }
      );
      return {
        resolved,
        unresolved,
        backlog,
        fields: ['index', 'code', 'text', 'edit'],
        lastActions: []
      };
    } catch (error) {
      console.log(
        `Couldn't get error lists:\n${error}\nDid you start the API?`
      );
      console.log(
        "HINT: You can comment out the full `asyncData` method and work with mocked data for UI/UX development, if you want to."
      );
    }
  },
  data() {
    return {
      resolved: [],
      unresolved: [],
      backlog: []
    };
  },
  methods: {
    unresolve (index, resolved, unresolved, lastActions){
      let item = resolved[index]
      
      // for undo button
      const lastAction = {index: index, sourceList: resolved, targetList: unresolved, item: $.extend(true, {}, item)}
      lastActions.push(lastAction)
      
      // unresolve
      item.text = item.text.replace("resolved", "unresolved") 
      unresolved.push(item)
      resolved.splice(index, 1)
    },
    resolve (index, unresolved, resolved, lastActions){
      let item = unresolved[index]

      // for undo button      
      const lastAction = {index: index, sourceList: unresolved, targetList: resolved, item: $.extend(true, {}, item)}
      lastActions.push(lastAction)

      // resolve
      item.text = item.text.replace("unresolved", "resolved")
      resolved.push(item)
      unresolved.splice(index, 1)
    },
    unresolveBacklog (index, backlog, unresolved, lastActions){
      let item = backlog[index]
            
      // for undo button
      const lastAction = {index: index, sourceList: backlog, targetList: unresolved, item: $.extend(true, {}, item)}
      lastActions.push(lastAction)
      
      // unresolve backlog
      item.text = item.text.replace("in the `backlog`", "`unresolved`")
      unresolved.push(item)
      backlog.splice(index, 1)
    },
    undo(lastActions){
      if (lastActions.length != 0){
        const lastAction = lastActions.pop()
        lastAction.targetList.pop()
        lastAction.sourceList.splice(lastAction.index, 0, lastAction.item)

      }
    }
  }
};
</script>
