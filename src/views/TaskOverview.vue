<template>
  <page-with-drawer :is-drawer-open="isDrawerOpen">
    <router-view :key="$route.fullPath" />
    <new-task-dialog v-model="newTaskDialog" />
    <div class="text-center my-8">
      <h1>
        Find tasks at
        <strong class="xr-title">
          Extinction Rebellion Nederland.
        </strong>
      </h1>
    </div>
    <div v-if="$vuetify.breakpoint.smAndDown" class="mb-8">
      <v-divider />
      <div class="d-flex justify-end pa-3">
        <new-item-button label="New Task" @click="showNewTaskDialog" />
        <v-btn text color="primary" @click="isDrawerOpen = true">
          Filter
        </v-btn>
      </div>
      <v-divider />
    </div>
    <grid-list v-if="filteredTasks.length > 0" gap="2rem">
      <task-card v-for="task in filteredTasks" :key="task.id" :task="task" />
    </grid-list>
    <div v-else class="pa-5 text-center">
      <h3>No results.</h3>
      <p>Try removing filters.</p>
    </div>
    <template v-slot:drawer>
      <default-drawer @close-drawer="handleCloseDrawer">
        <template #header>
          <div
            class="d-flex justify-space-between align-center"
            style="width:100%;"
          >
            <div class="d-flex flex-column">
              <span class="font-weight-bold">
                Search for tasks
              </span>
              <span class="font-weight-light">
                ({{ filteredTasks.length }} tasks found)
              </span>
            </div>
            <v-btn text color="primary">
              Clear filters
            </v-btn>
          </div>
        </template>
        <task-filters
          :on-set-filter="handleSelectFilter"
          :selected-filters="selectedFilters"
          :role-amount="filteredTasks.length"
        />
        <div v-if="!isMobile" class="text-center mt-4">
          <new-item-button label="New Task" @click="showNewTaskDialog" />
        </div>
      </default-drawer>
    </template>
  </page-with-drawer>
</template>
<script>
import TaskFilters from "@/components/tasks/TaskFilters.vue";
import DefaultDrawer from "@/components/layout/DefaultDrawer.vue";
import GridList from "@/components/layout/GridList.vue";
import TaskCard from "@/components/tasks/TaskCard.vue";
import PageWithDrawer from "@/components/layout/PageWithDrawer.vue";
import NewTaskDialog from "@/components/tasks/NewTaskDialog.vue";
import NewItemButton from "@/components/NewItemButton.vue";
import { mapGetters } from "vuex";
export default {
  name: "TasksOverview",
  components: {
    TaskFilters,
    PageWithDrawer,
    GridList,
    TaskCard,
    DefaultDrawer,
    NewTaskDialog,
    NewItemButton,
  },
  data: () => ({
    isDrawerOpen: null,
    newTaskDialog: false,
    selectedFilters: {
      text: "",
      localGroup: [],
      workingCircle: [],
    },
  }),
  computed: {
    ...mapGetters("tasks", ["getByFilters"]),
    filteredTasks: function() {
      return this.getByFilters(this.selectedFilters);
    },
    isMobile: function() {
      return this.$vuetify.breakpoint.smAndDown;
    },
  },
  watch: {
    isMobile: function() {
      this.isDrawerOpen = !this.isMobile;
    },
  },
  created: function() {
    this.isDrawerOpen = !this.isMobile;
  },
  methods: {
    handleSelectFilter: function(value, type) {
      this.selectedFilters[type] = value;
    },
    handleCloseDrawer: function() {
      this.isDrawerOpen = false;
    },
    showNewTaskDialog: function() {
      this.newTaskDialog = true;
    },
  },
};
</script>
