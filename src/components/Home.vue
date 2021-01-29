<template>
  <b-row style="margin-top: 10px">
    <b-col cols="8">
      <div>
        <b-nav tabs fill justified>
          <b-nav-item style="background: #0096c7;" @click="addTable()" :disabled="isEdit"><p class="text-white">Shto tavoline</p>
          </b-nav-item>
          <b-nav-item style="background: #ef476f;" v-if="!isEdit" @click="editTables()"><p class="text-white">Modifiko tavoline</p>
          </b-nav-item>
          <b-nav-item style="background: #16bf82;" v-else @click="doneEdit()"><p class="text-white">Perfundo modifikimet</p>
          </b-nav-item>
        </b-nav>
      </div>
      <b-card style="height: calc(100vh - 158px);">
        <div id="main" class="tableMain" v-for="item in tables">
          <VueDragResize :class="isEdit ? 'shake' : ''"
                         :sticks="[]"
                         :id="'tooltip-target-' + item.key"
                         class="tableStyles"
                         :isActive="isEdit"
                         :preventActiveBehavior="true" :w="80"
                         :parentLimitation="true"
                         :h="80" :y="item.top" :x="item.left" :z="0"
                         :parentH="1000"
                         :parentW="1200"
                         v-on:dragging="resize($event,item.key,item.tableName)">

            <b-icon icon="x-circle" v-if="isEdit" variant="danger" @click="removeTable(item.key)"
                    title="Fshi tavolinen"></b-icon>

            <div class="text-center" style="margin-top: 21px;">
              <h5 class="text-white">{{ item.tableName }}</h5>
            </div>
          </VueDragResize>
          <div v-if="isEdit">
            <b-popover :target="'tooltip-target-' + item.key" triggers="click" placement="top">
              <input class="form-control inputTableName" type="text" v-model="item.tableName"
                     placeholder="Emri i tavolines" @blur="tableNameChange(item.key,item.tableName)"/>
            </b-popover>
          </div>
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
      selectedProfile: 0,
      filteredTables: []
    }
  },

  methods: {
    resize(newRect, item) {
      let table = this.tables.find(x => x.key === item);
      table.top = newRect.top;
      table.left = newRect.left;
    },

    tableNameChange(key, tableName) {
      let tables = JSON.parse(localStorage.getItem('tables')).filter(x => x.tableName === tableName)
      if (tables.length > 0)
        alert('Tavolina me emrin ' + tableName + ' ekziston!')

      let table = this.tables.find(x => x.key === key);

      table.tableName = tableName;
    },
    addTable() {
      let localStorageItem = JSON.parse(localStorage.getItem('tables')).filter(x => x != null)
      let table = {
        key: Math.floor(Math.random() * 100) + 1,
        profileId: this.selectedProfile,
        top: Math.floor(Math.random() * 100) + 1,
        left: Math.floor(Math.random() * 100) + 1
      }
      this.tables = (localStorageItem !== null && localStorageItem.length > 0) ? localStorageItem : []
      this.tables.push(table)
      localStorage.setItem('tables', JSON.stringify(this.tables))
      this.selectProfile(this.selectedProfile)
    },
    editTables() {
      this.isEdit = true
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
              tableName: tableEntity.tableName,
              top: tableEntity.top,
              left: tableEntity.left,
              profileId: tableEntity.profileId
            }
          })

      localStorage.setItem('tables', JSON.stringify(unique))
      this.tables = JSON.parse(localStorage.getItem('tables')).filter(x => x.profileId === this.selectedProfile)
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
      this.selectProfile(this.selectedProfile)
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

.tableMain {
  padding-left: 149px;
  padding-top: 24px;
}

.tableStyles {
  box-shadow: 0 1px 6px 0 rgba(32, 33, 36, .28);
  background: #506dfb;
  border-radius: 20px;
  cursor: pointer
}

.inputTableName {
  background-color: transparent;
  border-radius: 25px;
}

.b-icon {
  float: right;
  cursor: pointer;
  margin-top: -14px;
}

.vdr.active:before {
  outline: none !important;
}

.tooltip > .tooltip-inner {
  background-color: transparent !important;
}

.shake {
  animation-delay: -.75s;
  animation-duration: .25s;
}

#main .vdr:nth-child(2n) {
  animation-name: keyframes1;
  animation-iteration-count: infinite;
  transform-origin: 50% 10%;
}

#main .vdr:nth-child(2n-1) {
  animation-name: keyframes2;
  animation-iteration-count: infinite;
  animation-direction: alternate;
  transform-origin: 30% 5%;
}

@keyframes keyframes1 {
  0% {
    transform: rotate(-1deg);
    animation-timing-function: ease-in;
  }

  50% {
    transform: rotate(1.5deg);
    animation-timing-function: ease-out;
  }
}

@keyframes keyframes2 {
  0% {
    transform: rotate(1deg);
    animation-timing-function: ease-in;
  }

  50% {
    transform: rotate(-1.5deg);
    animation-timing-function: ease-out;
  }
}
</style>
