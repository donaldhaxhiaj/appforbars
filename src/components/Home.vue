<template>
  <b-row style="margin-top: 10px">
    <b-col cols="8">
      <b-button v-if="isEdit" @click="doneEdit()">Done</b-button>
      <b-card style="height: calc(100vh - 100px);">
        <b-card-title>Tavolinat</b-card-title>
        <div v-for="item in tables" @dblclick="removeTable(item.key)">
          <VueDragResize :sticks="[]"
                         style="box-shadow: 0 1px 6px 0 rgba(32, 33, 36, .28); background: coral;border-radius: 20px;"
                         :isActive="isEdit" :preventActiveBehavior="true" :w="80"
                         :h="80" :y="item.top" :x="item.left" v-on:dragging="resize($event,item.key)">
            <div class="text-center">
              <h3>{{ item.key }}</h3>
              <p>{{ item.top }} х {{ item.left }} </p>
              <p>{{ item.width }} х {{ item.height }}</p>
            </div>
          </VueDragResize>
        </div>

      </b-card>
    </b-col>
    <b-col cols="4">
      <b-card style="height: calc(100vh - 100px);">
        <b-card-title>Konfigurime</b-card-title>
        <b-card-body>
          <div class="form-group">
            <b-button @click="showModal">Add profile</b-button>
            <b-button v-if="1===2" @click="addTable()">Add table</b-button>
          </div>
          <div class="form-group">
            <b-list-group>
              <b-list-group-item active-class="active" v-for="profile in profiles" @click="selectProfile(profile.Id)"
                                 style="cursor: pointer;">
                  <span>
                    {{ profile.name }}
                  </span>
                <div class="float-right">
                  <b-dropdown right text="Actions">
                    <b-dropdown-item @click="addTable(profile.Id)">Add table</b-dropdown-item>
                    <b-dropdown-item @click="editTables()">Edit tables</b-dropdown-item>
                  </b-dropdown>
                </div>
              </b-list-group-item>
            </b-list-group>
          </div>
        </b-card-body>
      </b-card>
    </b-col>
    <b-modal ref="profileModal" hide-footer title="Using Component Methods">
      <div class="d-block text-center">
        <b-form @submit.stop.prevent>
          <b-form-input :state="validation" id="feedback-user" v-model="profileName"
                        placeholder="Emri i profilit"></b-form-input>
          <b-form-invalid-feedback :state="validation">
            Emri i profilit duhet te jete 5-12 karaktere i gjate dhe te jete i ndryshem nga profilet e tjera.
          </b-form-invalid-feedback>
          <b-form-valid-feedback :state="validation">
            Eshte ne rregull.
          </b-form-valid-feedback>
        </b-form>
      </div>
      <b-button class="mt-3" variant="outline-danger" block @click="hideModal">Close</b-button>
      <b-button class="mt-2" variant="outline-warning" :disabled="validation === false" block @click="addProfile()">
        Save
      </b-button>
    </b-modal>
  </b-row>
</template>

<script>

import VueDragResize from 'vue-drag-resize'

export default {
  name: 'Home',
  components: {
    VueDragResize
  },
  data() {
    return {
      lastDeleted: 0,
      isEdit: false,
      profileName: '',
      profiles: [
        {Id: 1, name: "test"},
        {Id: 2, name: "test2"}
      ],
      tables: [],
      selectedProfile: 1,
      filteredTables: []
    }
  },

  methods: {
    resize(newRect, item) {
      let table = this.tables.find(x => x.key === item);
      table.top = newRect.top;
      table.left = newRect.left;
    },
    addTable(profileId) {
      let localStorageItem = JSON.parse(localStorage.getItem('tables'))
      let table = {
        key: (localStorageItem !== null && localStorageItem.length > 0) ? localStorageItem.length++ : 0,
        profileId: profileId,
        top: 100,
        left: 100
      }
      this.tables = (localStorageItem !== null && localStorageItem.length > 0) ? localStorageItem : []
      this.tables.push(table)
      localStorage.setItem('tables', JSON.stringify(this.tables))
    },
    editTables() {
      this.isEdit = 1
    },
    tablesJoining() {
      let self = this
      let tableFromStorage = JSON.parse(localStorage.getItem('tables')).filter(x => x !== null && this.lastDeleted !== x.key)
      if (tableFromStorage.length > 0) {
        tableFromStorage.forEach(function (item) {
          self.tables.push(item)
        })
      }
      let unique = Array.from(new Set(this.tables.map(x => x.key)))
          .map(key => {
            let tableEntity = this.tables.find(x => x.key === key);
            return {
              key: key,
              top: tableEntity.top,
              left: tableEntity.left,
              profileId: tableEntity.profileId
            }
          })
      console.log(unique)
      localStorage.setItem('tables', JSON.stringify(unique))
      this.tables = JSON.parse(localStorage.getItem('tables')).filter(x => x.profileId === 1)
    },

    doneEdit() {
      this.isEdit = !this.isEdit
      this.tablesJoining()
    },
    removeTable(tableKey) {
      this.tables = this.tables.filter(x => {
        return x.key !== tableKey
      })
      this.lastDeleted = tableKey
      this.tablesJoining()
    },
    addProfile() {
      if (this.profileName === '')
        return false
      this.profiles.push({Id: Math.floor(Math.random() * 20) + 1, name: this.profileName})
    },
    selectProfile(profileId) {
      let localStorageItem = JSON.parse(localStorage.getItem('tables')).filter(x => x !== null)
      this.selectedProfile = profileId
      this.tables = (localStorageItem !== null && localStorageItem.length > 0)
          ? localStorageItem.filter(x => x.profileId === this.selectedProfile) : []
    },
    hideModal() {
      this.$refs['profileModal'].hide()
    },
    showModal() {
      this.$refs['profileModal'].show()
    }
  },
  computed: {
    validation() {
      return this.profileName.length > 4 && this.profileName.length < 13 && this.profiles.some(x => x.name === this.profileName) === false
    }
  },

  created() {
    this.tables = JSON.parse(localStorage.getItem('tables')).filter(x => x.profileId === this.selectedProfile)
  }

}
</script>

<style scoped>
.vdr.active:before {
  outline: none !important;
}
</style>
