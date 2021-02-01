<template>
  <b-row style="margin-top: 10px">
    <b-card style="min-height: calc(100vh - 10px); min-width: 100%">
      <b-row style="min-height: 100%">
        <b-col cols="10">
          <div
            id="main"
            class="tableMain"
            v-for="(item, index) in profileTables"
            :key="index"
          >
            <VueDragResize
              :class="isMoving ? 'shake' : ''"
              :sticks="[]"
              :id="'tooltip-target-' + item.key"
              class="tableStyles"
              :isActive="isMoving"
              :w="90"
              :preventActiveBehavior="true"
              :parentLimitation="true"
              :h="90"
              :y="item.top"
              :x="item.left"
              :z="0"
              :parentH="1000"
              :parentW="1600"
              @dragstop="changePosition($event, item.key)"
            >
            <div style="position: absolute;top: -5px;right: 0">
              <b-icon
                  icon="arrows-move"
                  v-if="isEdit"
                  variant="info"
                  @click="isMoving = !isMoving"
                  title="Modifiko tavolinen"
                  class="ml-2 mb-2"
              ></b-icon>
              <b-icon
                  icon="pencil-square"
                  v-if="isEdit"
                  variant="info"
                  @click="isMoving = false;isNumberEdit = !isNumberEdit"
                  title="Modifiko tavolinen"
                  class="ml-2 mb-2"
              ></b-icon>
              <b-icon
                  icon="x-circle"
                  v-if="isEdit"
                  variant="danger"
                  @click="removeTable(item.key)"
                  title="Fshi tavolinen"
              ></b-icon>
            </div>

              <div v-if="isEdit" style="position: absolute;top: 15px;left: 21px;">
                <input :disabled="!isNumberEdit" class="form-control" v-model="item.number" style="font-size: 33px;color: white; background: transparent;width: 70%;height: 57px;"/>
              </div>
              <div v-else class="text-center">
                <h1  class="text-white" style="margin-top: 1rem;">{{ item.number }}</h1>
              </div>
            </VueDragResize>
          </div>
        </b-col>
        <b-col cols="2">
          <b-row style="min-height: 100%">
            <b-col cols="12" class="align-self-center">
              <b-card
                v-for="(profile, index) in profiles"
                :key="index"
                class="mb-3 text-center"
                :class="getBorderClass(profile.id)"
                @click="selectProfile(profile.id)"
                style="cursor: pointer"
              >
                <h4>{{ profile.name }}</h4>
              </b-card>
            </b-col>
            <b-col cols="12" class="align-self-end">
              <div class="form-group">
                <b-button v-show="!isEdit" pill block id="add-plane-button"
                  >Shto Planimetri</b-button
                >

                <b-popover
                  target="add-plane-button"
                  triggers="click"
                  ref="plane-tooltip"
                  placement="left"
                >
                  <div class="d-flex">
                    <input
                      class="form-control inputTableName mr-3"
                      type="text"
                      v-model="profileName"
                      placeholder="Emri i planit"
                    />
                    <b-button
                      pill
                      size="sm"
                      variant="primary"
                      @click="addPlane()"
                      >Krijo</b-button
                    >
                  </div>
                </b-popover>
                <b-button
                  block
                  pill
                  variant="primary"
                   @click="editTables()"
                  v-show="!isEdit"
                  >Modifiko Planimetrin</b-button
                >
                <b-button
                  block
                  pill
                  v-show="isEdit"
                  id="add-table-button"
                  >Shto Tavoline</b-button
                >
                <b-popover
                  target="add-table-button"
                  triggers="click"
                  ref="table-tooltip"
                  placement="left"
                >
                  <div class="d-flex">
                    <input
                      class="form-control inputTableName mr-3"
                      type="number"
                      v-model="tableNumber"
                      placeholder="Numri i tavolines"
                    />
                    <b-button
                      pill
                      size="sm"
                      variant="primary"
                     @click="addTable"
                      >Shto</b-button
                    >
                  </div>
                </b-popover>
                <b-button
                  block
                  pill
                  v-if="isEdit"
                  variant="success"
                  @click="doneEdit()"
                  >Perfundo modifikimet</b-button
                >
              </div>
            </b-col>
          </b-row>
        </b-col>
      </b-row>
    </b-card>
  </b-row>
</template>

<script>
import VueDragResize from "vue-drag-resize";

export default {
  name: "TableManager",
  components: {
    VueDragResize,
  },
  created() {
    this.profiles = JSON.parse(localStorage.getItem("ordis:planes")) || [];
    if (this.profiles.length) {
      this.selectedProfile = this.profiles.find((profile) => profile.default);
      if (!this.selectedProfile) {
        this.selectedProfile = this.profiles.sort((a, b) => a.id - b.id)[0].id;
      }
      this.tables = JSON.parse(localStorage.getItem("ordis:tables")) || [];
    }
  },
  data() {
    return {
      lastDeleted: 0,
      isEdit: false,
      profileName: "",
      tableNumber: "",
      profiles: [],
      tables: [],
      selectedProfile: 0,
      filteredTables: [],
      isMoving:false,
      isNumberEdit:false
    };
  },
  methods: {
    getBorderClass(planeId) {
      return this.selectedProfile === planeId
        ? "border border-3 border-primary"
        : "";
    },
    changePosition(newPosition, id) {
      let index = this.tables.indexOf((table) => table.key === id);
      let table = this.tables.find((x) => x.key === id);
      table.top = newPosition.top;
      table.left = newPosition.left;
      this.tables.slice(index, 1, table);
      localStorage.setItem("ordis:tables", JSON.stringify(this.tables));
    },
    tableNameChange(key, tableName) {
      let tables = JSON.parse(localStorage.getItem("ordis:tables")).filter(
        (x) => x.tableName === tableName
      );
      if (tables.length > 0)
        alert("Tavolina me emrin " + tableName + " ekziston!");

      let table = this.tables.find((x) => x.key === key);

      table.tableName = tableName;
    },
    addTable() {
      let table = {
        key: Math.floor(Math.random() * 100) + 1,
        profileId: this.selectedProfile,
        top: Math.floor(Math.random() * 100) + 1,
        left: Math.floor(Math.random() * 100) + 1,
        number: this.tableNumber,
      };
      this.tables.push(table);
      this.tableNumber = "";
      this.$refs["table-tooltip"].$emit("close");
      localStorage.setItem("ordis:tables", JSON.stringify(this.tables));
    },
    editTables() {
      this.isEdit = true;
    },
    doneEdit() {
      this.isEdit = false;
      this.isMoving = false;
    },
    removeTable(tableKey) {
      this.tables = this.tables.filter((x) => {
        return x.key !== tableKey;
      });
      this.lastDeleted = tableKey;
      localStorage.setItem("ordis:tables", JSON.stringify(this.tables));
    },
    addPlane() {
      if (this.profileName === "") return false;
      this.profiles.push({
        id: Math.floor(Math.random() * 20) + 1,
        name: this.profileName,
      });
      // Close Plane Tooltip
      this.$refs["plane-tooltip"].$emit("close");
      // Empty profile name value
      this.profileName = "";
      // Handle case when no plane exists
      // Set newly created one as default
      if (!this.selectedProfile) {
        this.selectedProfile = this.profiles[0].id;
      }
      // Save in localstorage
      localStorage.setItem("ordis:planes", JSON.stringify(this.profiles));
    },
    selectProfile(profileId) {
      this.selectedProfile = profileId;
    },
    hideModal() {
      this.$refs["profileModal"].hide();
    },
    showModal() {
      this.$refs["profileModal"].show();
    }
  },
  computed: {
    profileTables() {
      return (
        this.tables.filter(
          (table) => table.profileId === this.selectedProfile
        ) || []
      );
    },
  },
};
</script>

<style scoped>
.tableMain {
  padding-left: 149px;
  padding-top: 24px;
}

.tableStyles {
  box-shadow: 0 1px 6px 0 rgba(32, 33, 36, 0.28);
  background: #6e6d6d;
  border-radius: 20px;
  cursor: pointer;
}

.border-3 {
  border-width: 3px !important;
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
  animation-delay: -0.75s;
  animation-duration: 0.25s;
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
