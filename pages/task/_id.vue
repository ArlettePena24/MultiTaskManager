<template>
  <div>
    <v-btn nuxt to="/"> Go back </v-btn>
    <v-card max-width="600" min-width="300" class="mx-auto pa-5" p4 v-if="data">
      <v-card-item>
        <v-card-title
          ><h2 :style="{ color: '#9FA8DA' }">
            Id: {{ data[0].id }}
          </h2></v-card-title
        >
        <v-card-subtitle>
          <p :style="{ color: '#CFD8DC' }">Title: {{ data[0].title }}</p>
          <p v-if="data[0].is_completed !== 0" :style="{ color: '#C8E6C9' }">
            Completed
          </p>
          <p v-else :style="{ color: '#FBE9E7' }">
            Not completed
          </p></v-card-subtitle
        >
        <v-card-text>
          <p v-if="data[0].due_date" :style="{ color: '#CFD8DC' }">
            Due Date: {{ data[0].due_date }}
          </p>
          <p v-else :style="{ color: '#CFD8DC' }">Due Date not provided</p>

          <p v-if="data[0].description" :style="{ color: '#CFD8DC' }">
            Description: {{ data[0].description }}
          </p>
          <p v-else :style="{ color: '#CFD8DC' }">
            This task has no description
          </p>

          <p v-if="data[0].comments" :style="{ color: '#CFD8DC' }">
            Comments: {{ data[0].comments }}
          </p>
          <p v-else :style="{ color: '#CFD8DC' }">This task has no comments</p>

          <p v-if="data[0].tags" :style="{ color: '#CFD8DC' }">
            Tags: {{ data[0].tags }}
          </p>
          <p v-else :style="{ color: '#CFD8DC' }">This task has no tags</p>
        </v-card-text>
      </v-card-item>

      <v-card-actions>
        <v-btn color="#C8E6C9" @click="overlay = !overlay">Edit Task</v-btn>
        <v-btn color="#283593" @click="deleteTask">Delete Task</v-btn>
      </v-card-actions>
    </v-card>
    <div>
      <v-overlay v-model="overlay">
        <v-form
          @submit.prevent="editTask"
          fast-fail
          style="background-color: rgba(23, 23, 23, 1)"
          class="pa-7"
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
  </div>
</template>

<script>
import axios from 'axios'
export default {
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
      },
      overlay: false,
    }
  },
  async asyncData({ params }) {
    const token =
      'e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd'

    try {
      const response = await axios.get(
        `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${params.id}`,
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
  computed: {
    isFormValid() {
      const { title, is_completed } = this.formData
      return title && is_completed !== null
    },
  },

  methods: {
    refresh() {
      this.$nuxt.refresh()
    },
    async editTask() {
      const token =
        'e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd'

      const taskId = this.data[0].id
      console.log(this.formData)
      try {
        const response = await axios.put(
          `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${taskId}`,
          this.formData,
          {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          }
        )
        this.overlay = !this.overlay
        console.log(response.data)
        this.refresh()
      } catch (error) {
        console.error(error)
      }
    },
    async deleteTask() {
      const token =
        'e864a0c9eda63181d7d65bc73e61e3dc6b74ef9b82f7049f1fc7d9fc8f29706025bd271d1ee1822b15d654a84e1a0997b973a46f923cc9977b3fcbb064179ecd'

      const taskId = this.data[0].id

      try {
        await axios.delete(
          `https://ecsdevapi.nextline.mx/vdev/tasks-challenge/tasks/${taskId}`,
          {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          }
        )
        this.$router.push('/')
      } catch (error) {
        console.error(error)
      }
    },
  },
}
</script>
