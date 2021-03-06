<template>
  <router-link
    class="session-item"
    :class="cssClass"
    :to="{ name: 'session', params: { sessionId: session.id } }"
  >
    <!-- Author Avatar -->
    <div class="avatar">
      <img
        v-if="session.user.avatar"
        class="img"
        :src="session.user.avatar"
      >
    </div>

    <!-- Session content -->
    <div class="content">
      <div class="header">
        <span class="title">{{ session.title }}</span>

        <span v-if="!session.public" class="tag private">private</span>
      </div>

      <div class="info">
        <span class="author">{{ session.user.name }}</span>
        <span v-if="session.user.email" class="email">{{ session.user.email }}</span>
        <span class="date">
          <BaseIcon icon="schedule"/>
          <BaseTimeAgo :date="session.date"/>
        </span>
      </div>
    </div>

    <!-- Actions -->
    <div v-if="user && !hideActions" class="actions">
      <template v-if="user.admin">
        <BaseButton
          :icon="session.public ? 'lock_open' : 'lock'"
          class="icon-button secondary"
          title="Toggle public"
          :class="{
            selected: session.public,
          }"
          @click.prevent.stop="togglePublic"
        />

        <BaseButton
          icon="delete"
          class="icon-button secondary"
          @click.prevent.stop="removeSession"
        />
      </template>
    </div>
  </router-link>
</template>

<script>
import { mapGetters } from 'vuex'
import { caheSessionRemove } from '../cache/sessions'

import SESSION_TOGGLE_PUBLIC_MUTATION from '../graphql/SessionTogglePublic.gql'
import SESSION_REMOVE_MUTATION from '../graphql/SessionRemove.gql'
import SESSIONS_ALL_QUERY from '../graphql/SessionsAll.gql'
import SESSIONS_USER_QUERY from '../graphql/SessionsUser.gql'

export default {
  props: {
    session: {
      type: Object,
      required: true,
    },

    hideActions: {
      type: Boolean,
      default: false,
    },
  },

  computed: {
    ...mapGetters('user', [
      'user',
    ]),

    cssClass () {
      return {
        private: !this.session.public,
      }
    },
  },

  methods: {
    togglePublic () {
      this.$apollo.mutate({
        mutation: SESSION_TOGGLE_PUBLIC_MUTATION,
        variables: {
          id: this.session.id,
        },
        optimisticResponse: {
          __typename: 'Mutation',
          sessionTogglePublic: {
            __typename: 'Session',
            id: this.session.id,
            public: !this.session.public,
          },
        },
      })
    },

    removeSession () {
      if (!confirm('Delete this session?')) return

      this.$apollo.mutate({
        mutation: SESSION_REMOVE_MUTATION,
        variables: {
          id: this.session.id,
        },
        update: (store, { data: { sessionRemove } }) => {
          caheSessionRemove(store, [
            {
              query: SESSIONS_ALL_QUERY,
              dataProp: 'sessionsAll',
            },
            {
              query: SESSIONS_USER_QUERY,
              dataProp: 'sessionsUser',
            },
          ], sessionRemove)
        },
        optimisticResponse: {
          __typename: 'Mutation',
          sessionRemove: this.session,
        },
      })
    },
  },
}
</script>

<style lang="stylus" scoped>
@import "../styles/imports"

.session-item
  display block
  padding 24px 32px
  h-box()
  align-items stretch
  border-radius 3px

  @media (max-width: $small-screen)
    padding 12px

  .content
    margin 4px 0
    margin-right 24px
    flex auto 1 1
    width 0

    @media (max-width: $small-screen)
      margin-right 12px

    .title
      font-size 1.4em
      font-weight lighter

    .text
      white-space pre-pre-wrap
      margin-bottom 6px

    .title,
    .text
      word-wrap break-word

  .info
    color rgba($md-white, .5)
    font-size 0.8em

    >>> > span
      margin-right 12px

      .icon
        margin-right 4px

  .avatar
    margin-right 24px
    margin-top 6px

    @media (max-width: $small-screen)
      margin-right 12px

  .header
    h-box()
    align-items center

  .tag
    &.private
      background $color-accent
      color $md-black

  .actions
    h-box()
    box-center()

  &:hover
    background $color-secondary

  &.private
    .avatar
      opacity .5
</style>
