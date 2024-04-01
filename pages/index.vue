<template>
  <div>
    <v-btn @click="overlay = !overlay" color="#283593"> Add a New Task</v-btn>
    <v-card
      max-width="600"
      min-width="300"
      class="mx-auto pa-5 ma-5"
      v-for="task in data"
      :key="task.id"
    >
      <v-card-item>
        <v-card-title>
          <p>Tarea Id. {{ task.id }}</p>
        </v-card-title>
        <v-card-subtitle>
          <p v-if="task.due_date" :style="{ color: '#CFD8DC' }">
            Due Date: {{ task.due_date }}
          </p>
          <p v-else :style="{ color: '#CFD8DC' }">Due Date not provided</p>

          <v-card-item>
            <p :style="{ color: '#CFD8DC' }">Title: {{ task.title }}</p>
            <p v-if="task.is_completed !== 0" :style="{ color: '#C8E6C9' }">
              Completed
            </p>
            <p v-else :style="{ color: '#FBE9E7' }">Not completed</p>
          </v-card-item>
        </v-card-subtitle>
      </v-card-item>
      <v-card-actions>
        <v-btn nuxt :to="`/task/${task.id}`" color="#C8E6C9"> Details </v-btn>
      </v-card-actions>
    </v-card>
    <v-overlay v-model="overlay">
      <v-form
        @submit.prevent="addTask"
        fast-fail
        style="background-color: rgba(23, 23, 23, 1)"
        class="pa-8"
      >
        <v-text-field
          v-model="formData.title"
          label="Title"
          required
          :rules="[
            (v) => !!v || 'Title is required',
            (v) =>
              /^[a-zA-Z\s]*$/.test(v) ||
              'Title must contain only letters and spaces',
          ]"
        ></v-text-field>

        <v-text-field
          v-model="formData.is_completed"
          label="Is Completed"
          required
          :rules="[
            (v) => !!v || 'Is Completed is required',
            (v) =>
              /^(0|1)$/.test(v) ||
              'Is Completed must be 0(not done) or 1 (done)',
          ]"
        ></v-text-field>

        <v-text-field
          v-model="formData.due_date"
          label="Due Date"
          :rules="[
            (v) =>
              !v ||
              /^\d{4}-\d{2}-\d{2}$/.test(v) ||
              'Due Date must be in YYYY-MM-DD format',
          ]"
        ></v-text-field>

        <v-text-field
          v-model="formData.description"
          label="Description"
        ></v-text-field>

        <v-text-field
          v-model="formData.comments"
          label="Comments"
        ></v-text-field>

        <v-text-field v-model="formData.tags" label="Tags"></v-text-field>
        <v-btn @click="overlay = !overlay" class="mt-2" color="#283593"
          >Cancel</v-btn
        >
        <v-btn
          color="#C8E6C9"
          class="mt-2"
          type="submit"
          :disabled="!isFormValid"
          block
          >Submit</v-btn
        >
      </v-form>
    </v-overlay>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  watchQuery: true,
  data() {
    return {
      formData: {
        title: '',
        is_completed: '',
        due_date: `${new Date().getFullYear()}-${
          new Date().getMonth() + 1 < 10
            ? '0' + (new Date().getMonth() + 1)
            : new Date().getMonth() + 1
        }-${
          new Date().getMonth() < 10
            ? '0' + new Date().getDate()
            : new Date().getDate()
        }`,
        description: '',
        comments: '',
        tags: '',
        token: '',
      },
      overlay: false,
    }
  },

  async asyncData({}) {
    const token =
      'e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd'

    try {
      const response = await axios.get(
        `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks`,
        {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        }
      )
      return {
        data: response.data,
        error: null,
      }
    } catch (error) {
      return {
        data: null,
        error: error.message,
      }
    }
  },
  methods: {
    refresh() {
      this.$nuxt.refresh()
    },
    async addTask() {
      const token =
        'e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd'

      try {
        const response = await axios.post(
          'https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks',
          this.formData,
          {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          }
        )
        console.log(response)
        this.overlay = !this.overlay
        this.refresh()
      } catch (error) {
        console.error(error)
      }
    },
  },
  computed: {
    isFormValid() {
      const { title, is_completed } = this.formData
      return title && is_completed !== null
    },
  },
}
</script>
