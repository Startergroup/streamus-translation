<template>
  <Sidebar
    :is-open="isOpen"
    :is-bottom="true"
    :width="getCommentsWidth"
    @update:is-open="$emit('update:comments', $event)"
  >
    <template #default>
      <div class="flex flex-col justify-start items-start w-full md:px-8 md:py-6 p-4 h-full overflow-hidden relative">
        <h1
          class="text-xl font-bold text-primary mb-6"
          @click="hideComment"
        >
          {{ $t("message.questionTitle") }}
        </h1>

        <div class="flex flex-col w-full items-start">
          <textarea
            v-model="comment"
            :class="[
              'textarea mb-8',
              { 'comment_error' : isEmptyTextarea }
            ]"
            @focus="isEmptyTextarea = false"
            @keyup.ctrl.enter.prevent="newLine($event)"
            @keydown.enter.prevent.exact="sendComment"
          ></textarea>

          <Button
            classes="button_primary w-28"
            :value="$t('message.questionButton')"
            @on-click="sendComment"
          />
        </div>

        <div v-show="successfully" class="absolute left-0 top-0 w-full h-full bg-gray-900 opacity-80"></div>

        <div
          :class="[
            'flex justify-center items-center absolute left-0 right-0 top-0 bottom-0 m-auto w-80 h-12 rounded-lg bg-green-500 transition transition-all duration-500 successfully_hide',
            { 'successfully_show' : successfully }
          ]"
        >
          <span class="text-base font-semibold text-white">
            {{ $t("message.questionMessage") }}
          </span>
        </div>
      </div>
    </template>
  </Sidebar>
</template>

<script>
import Button from '@/components/UI/Button.vue'
import Sidebar from '@/components/UI/Sidebar.vue'

import { getDatabase, ref, set } from 'firebase/database'
import { v4 as uuidv4 } from 'uuid'
import { mapGetters } from 'vuex'

export default {
  name: 'Comments',
  components: {
    Button,
    Sidebar
  },
  props: {
    isOpen: {
      type: Boolean,
      required: true
    }
  },
  data: () => {
    return {
      isEmptyTextarea: false,
      comment: null,
      successfully: false
    }
  },
  computed: {
    ...mapGetters([
      'currentTab'
    ]),
    getCommentsWidth () {
      return window.innerWidth > 1024 ? '50%' : '95%'
    }
  },
  methods: {
    sendComment () {
      if (!this.comment) {
        this.isEmptyTextarea = !this.comment

        return false
      }

      if (!this.currentTab) {
        return alert(this.$t('message.errorComments'))
      }

      const db = getDatabase()
      const date = new Date().toLocaleString('ru-RU', { timeZone: 'Europe/Moscow' })

      set(ref(db, `${this.currentTab.name}/${uuidv4()}`), {
        comment: this.comment,
        date: date
      })

      this.showSuccessMessage()

      this.comment = null
    },
    hideComment () {
      this.isEmptyTextarea = false
    },
    showSuccessMessage () {
      this.successfully = true
      setTimeout(() => {
        this.successfully = false
        this.$emit('update:comments', false)
      }, 1500)
    },
    newLine (event) {
      const caret = event.target.selectionStart

      event.target.setRangeText('\n', caret, caret, 'end')
      this.comment = event.target.value
    }
  }
}
</script>
