<template>
  <div
    class="menu"
    @click="onOpenMenuHandler"
  >
    <IconBase
      name="menu"
      class="menu__icon"
      :view-box-size="[24, 16]"
      :width="24"
      :height="16"
      fill="#6C63FF"
    />
  </div>

  <Sidebar
    :is-open="isOpen"
    width="300px"
    @update:is-open="isOpen = $event"
  >
    <template #default>
      <div class="flex flex-col w-full py-10">
        <h2 v-html="getTitle" class="text-xl font-bold text-black text-center"></h2>

        <div class="flex flex-col mt-10 pl-6 w-full">
          <Button
            v-for="(item, index) in getSortedTabs"
            :key="index"
            :classes="[
              'button button_empty rounded-l-3xl rounded-r-none justify-start pl-5',
              { 'menu__button_active': selectedTabId === item.tab_id }
            ]"
            @click="selectTabHandler(item.tab_id)"
          >
            <template #content>
              {{ item.name }}
            </template>
          </Button>
        </div>

        <div class="flex flex-col mt-10 pl-6 w-full">
          <Button
            v-for="(item, index) in actions"
            :key="index"
            :classes="['menu__button button button_empty rounded-l-3xl rounded-r-none justify-start pl-5 gap-4']"
            @click="item.onClick"
          >
            <template #content>
              <icon-base v-bind="item.icon" />
              {{ item.value }}
            </template>
          </Button>
        </div>

        <div class="flex flex-col mt-10 pl-6 w-full">
          <Button
            classes="button font-semibold rounded-l-3xl rounded-r-none justify-start pl-5 hover:bg-red-400 hover:text-white"
            :value="$t('message.logout')"
            @on-click="logoutHandler"
          />
        </div>
      </div>
    </template>
  </Sidebar>
</template>

<script>
import Button from '@/components/UI/Button.vue'
import IconBase from '@/components/Icons/IconBase.vue'
import Sidebar from '@/components/UI/Sidebar.vue'
import VoteModal from '@/components/Modals/VoteModal.vue'

import { mapActions, mapGetters, mapMutations, mapState } from 'vuex'
import { sessionStorageNames } from '@/constants/sessionStorage'

export default {
  name: 'Menu',
  components: {
    Button,
    IconBase,
    Sidebar
  },
  emits: ['open:quiz', 'open:schedule', 'open:comments'],
  data () {
    return {
      isOpen: false
    }
  },
  computed: {
    ...mapGetters([
      'getSortedTabs',
      'currentTab'
    ]),
    ...mapState([
      'tabs',
      'selectedTabId',
      'settings'
    ]),
    ...mapState('auth', [
      'user'
    ]),
    ...mapState('quiz', [
      'quizzes'
    ]),
    ...mapState('vote', ['votes']),
    getTitle () {
      if (!this.settings) return ''

      const currentLocale = this.settings[`title_${this.$i18n.locale}`]
      const str = currentLocale.replace(/[0-9]/g, '').replace(/[^a-zа-яё0-9\s]/gi, '')
      const num = parseInt(currentLocale.match(/\d+/)) || ''

      return `${str}<span class="text-primary">/</span> ${num}`
    },
    actions () {
      return [
        {
          hide: !this.quizzes.every(item => item.is_active) || !this.quizzes.length,
          value: 'Квиз',
          icon: {
            name: 'quiz',
            width: '23',
            height: '24',
            strokeColor: '#454545',
            strokeWidth: '1.5',
            strokeLinecap: 'round',
            strokeLinejoin: 'round',
            viewBoxSize: [23, 24]
          },
          onClick: () => {
            this.$emit('open:quiz', true)
          }
        },
        {
          hide: this.user.isGuest,
          value: 'Задать вопрос',
          icon: {
            name: 'question',
            width: '23',
            height: '24',
            strokeColor: '#454545',
            strokeWidth: '1.5',
            strokeLinecap: 'round',
            strokeLinejoin: 'round',
            viewBoxSize: [23, 24]
          },
          onClick: () => {
            this.$emit('open:comments', true)
          }
        },
        {
          hide: !Object.hasOwn(this.votes, 'presentations'),
          value: 'Голосование',
          icon: {
            name: 'survey',
            width: '23',
            height: '24',
            strokeColor: '#454545',
            strokeWidth: '1.5',
            strokeLinecap: 'round',
            strokeLinejoin: 'round',
            viewBoxSize: [23, 24]
          },
          onClick: () => {
            if (!this.selectedTabId) {
              return alert('Сначала надо выбрать секцию.')
            }

            this.$vfm.show({
              component: VoteModal
            })
          }
        }
      ].filter(item => !item.hide)
    }
  },
  methods: {
    ...mapActions('quiz', ['getQuizzes']),
    ...mapActions('auth', [
      'logout'
    ]),
    ...mapActions('vote', ['getVotes']),
    ...mapMutations([
      'setTabs',
      'selectTab'
    ]),
    ...mapMutations('quiz', [
      'setQuizzes'
    ]),
    ...mapMutations('vote', ['setVotes']),
    logoutHandler () {
      this.logout()
      this.$router.push('/auth')
    },
    async onOpenMenuHandler () {
      const quizzes = await this.getQuizzes(this.user.code_id)
      this.setQuizzes(quizzes)

      if (!this.user.isGuest) {
        const votes = await this.getVotes({ tabID: this.selectedTabId, userID: this.user.code_id })
        this.setVotes(votes)
      }

      this.isOpen = true
    },
    async selectTabHandler (id) {
      sessionStorage.setItem(sessionStorageNames.selectedTab, JSON.stringify(id))

      this.selectTab(id)
      this.isOpen = false
    }
  }
}
</script>
