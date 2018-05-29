<template lang="pug">

  .widget.notice
    ul.info-bar
      li.info(v-for='site in sites' @click='currentSite = site' :class='{ selected: currentSite == site }')
        .title {{ site }}
    ul.detail-list
      li(v-for='item in filteredNotice' :key='item.title' :class='{ important: item.isImportant }')
        a(:href='viewLink(item)')
          .top
            .left
              .tag.important(v-if='item.isImportant') 重要
              .tag.attachment(v-if='item.isAttachment') 附件
              span {{ item.title }}
            .right {{ formatDateNatural(item.time) }}
          .bottom(v-if='item.category')
            .left {{ item.category }}
    .empty(v-if='!filteredNotice || !filteredNotice.length') 暂无通知

</template>
<script>

  import H from '@/api'
  import formatter from '@/util/formatter'
  import markdown from '@/components/Markdown'

  export default {
    props: ['user'],
    components: { markdown },
    data() {
      return {
        notice: [],
        markdown: '',
        currentSite: null,
      }
    },
    persist: {
      notice: 'herald-default-notice'
    },
    created() {
      this.reload()
      if (/[?&]nid=(\d+)/.test(window.location.search)) {
        this.loadMarkdown({ nid: RegExp.$1 })
      }
    },
    watch: {
      user () {
        this.reload()
      }
    },
    computed: {
      sites() {
        return this.notice.map(k => k.site)
          .sort().filter((k, i, a) => a.indexOf(k) === i)
          .sort((a, b) => {
            let predef = ['教务处', '总务处', 'SRTP'].reverse()
            a = ~predef.indexOf(a)
            b = ~predef.indexOf(b)
            return a - b
          })
          .reduce((a, b) => a.slice(-1)[0] === b ? a : a.concat(b), [])
      },
      filteredNotice() {
        if (this.sites.length === 0) {
          return []
        }
        if (!this.currentSite) {
          this.currentSite = this.sites[0]
        }
        return this.notice.filter(k => k.site === this.currentSite)
      }
    },
    methods: {
      ...formatter,
      async reload() {
        let notice = await H.api.notice()
        let competition = await H.api.srtp.competition()
        this.notice = notice.concat(competition.map(k => ({
          title: k.name,
          site: 'SRTP',
          time: k.startTime,
          srtpId: k.id
        }))).sort((a, b) => b.time - a.time)
      },
      viewLink(notice) {
        if (notice.isAttachment) {
          return notice.url
        } else if (notice.site === 'SRTP') {
          return '#/notice/competition/' + notice.srtpId
        } else if (notice.nid != null) {
          return '#/notice/' + notice.nid
        } else {
          return '#/notice/url/' + encodeURIComponent(notice.url)
        }
      }
    }
  }

</script>
<style lang="stylus">
  .widget.notice
    li.info
      cursor pointer
      transition .3s

      &:not(.selected)
        background none
        color var(--color-text-bold)

        .title
          font-weight normal

    .important .top .left
      font-weight bold

    .tag
      display inline-block
      border-radius 3px
      margin-right 5px
      color #ffffff
      font-size 13px
      padding 1px 3px
      vertical-align baseline
      background var(--color-primary)

      &.important
        font-weight bold

</style>