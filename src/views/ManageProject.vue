<script>
  import FirebaseService from '@/api/firebase-service.js';
  import ProjectHeader from '@/components/ProjectHeader';

  import criteria from '@/data/criteria.json';

  export default {
    name: 'ManageProject',

    // Template dependencies
    components: {
      ProjectHeader
    },

    // Local state
    data: () => ({
      criteria: []
    }),

    computed: {
      project() {
        return this.$store.getters.getProject(this.$route.params.id);
      },

      latestSprint() {
        return _.findLast(_.orderBy(this.project.sprints, "sprintNumber", "asc"));
      }
    },

    // Events
    created() {
      this.$store.commit('initialiseCriteria', criteria);
    },

    mounted() {
      this.$router.push({ name: 'ManageTeamHealth' });
    },

    // Non-Reactive properties
    methods: {
      closeProjectTab() {
        this.$emit('closeTab', { id: this.$route.params.id });
        },

      addSprint() {
        const sprint = { teams: [] };
        _.forEach(this.project.teams, (team) => {
          _.forEach(this.criteria, (criterion) => {
            sprint.teams[team][criterion] = {
              value: 0
            }
          });
        });
      },

      getCriteria() {
        FirebaseService.getCriteria().then((data) => this.criteria = data);
      },

      reset() {
        this.sprints = JSON.parse(sessionStorage.getItem(`${this.project.id}.sprints`));
      },

      updateTeams(team) {
        // Create teams property if it doesn't exist
        if (!this.project.teams) {
          this.project.teams = [];
        }
        let teamKey = _.camelCase(team);
        this.project.teams[teamKey] = team;
      },

      hasProjectData() {
        return !_.isEmpty(this.project.teams) && !_.isEmpty(this.project.sprints);
      }
    }
  }
</script>

<template>
  <section>
    <ProjectHeader v-if="project"></ProjectHeader>
    <router-view></router-view>
    <router-view name="ManageTeams">
    </router-view>
    <router-view name="manageSprints"></router-view>
  </section>
</template>