<script>
import { isActive, hashRE, groupHeaders } from './util'

export default {
  functional: true,
  props: ['item'],
  render (h, { parent: { $page, $site, $route }, props: { item }}) {
    // console.log(">>>>>>>>>>>>>>>>>>>>>>>>",item);
    let tag = null
    if(item.title.search('#new')!=-1){
      tag = 'new'
    } else if (item.title.search('#update')!=-1){
      tag = 'update'
    }
    // console.log("$page>",$page.path);
    // console.log("item>>",item.path);
    // use custom active class matching logic
    // due to edge case of paths ending with / + hash
    const selfActive = isActive($route, item.path)
    // for sidebar: auto pages, a hash link should be active if one of its child
    // matches
    const active = item.type === 'auto'
      ? selfActive || item.children.some(c => isActive($route, item.basePath + '#' + c.slug))
      : selfActive
    const link = renderLink(h, item.path, item.title || item.path, active, tag)
    const configDepth = $page.frontmatter.sidebarDepth != null
      ? $page.frontmatter.sidebarDepth
      : $site.themeConfig.sidebarDepth
    const maxDepth = configDepth == null ? 1 : configDepth
    if (item.type === 'auto') {
      return [link, renderChildren(h, item.children, item.basePath, $route, maxDepth)]
    } else if (active && item.headers && !hashRE.test(item.path)) {
      const children = groupHeaders(item.headers)
      return [link, renderChildren(h, children, item.path, $route, maxDepth)]
    } else {
      return link
    }
  }
}

function renderLink (h, to, text, active, activeTag) {
  let textx = activeTag?text.replace('<!--#'+activeTag+'-->',''):text
  return h('router-link', {
    props: {
      to,
      activeClass: '',
      exactActiveClass: '',
    },
    class: {
      active,
      'sidebar-link': true
    }

  }, [textx,activeTag?h('span',{class: 'span-new-update'},activeTag):null])
}

function renderChildren (h, children, path, route, maxDepth, depth = 1) {
  if (!children || depth > maxDepth) return null
  return h('ul', { class: 'sidebar-sub-headers' }, children.map(c => {
    const active = isActive(route, path + '#' + c.slug)
    return h('li', { class: 'sidebar-sub-header' }, [
      renderLink(h, '#' + c.slug, c.title, active),
      renderChildren(h, c.children, path, route, maxDepth, depth + 1)
    ])
  }))
}
</script>

<style lang="stylus">
@import './styles/config.styl'
.span-new-update
  position: absolute;
  right: 5px;
  background: $accentColor
  color: rgb(255, 255, 255);
  font-size: 10px;
  font-weight: lighter !important;
  border-radius: 5px;
  padding-bottom: 2px;
  padding-left: 4px;
  padding-right: 4px;
  line-height: 15px;
  top: 50%;
  transform: translate(0,-50%);
  text-transform: capitalize;
.sidebar-sub-header
  a
    opacity: 1 !important;
.sidebar .sidebar-sub-headers
  padding-left 1rem
  font-size 0.95em
a.sidebar-link
  position: relative;
  font-weight 400
  display inline-block
  color $textColor
  border-left 0.25rem solid transparent
  padding 0.35rem 1rem 0.35rem 1.25rem
  line-height 1.4
  width: 100%
  box-sizing: border-box
  opacity: .7;
  transition: all .3s ease;
  &:hover
    color $accentColor
    opacity: 1 !important;
  &.active
    font-weight 600
    color $accentColor
    border-left-color $accentColor
    opacity: 1 !important;
  .sidebar-group &
    padding-left 2rem
  .sidebar-sub-headers &
    padding-top 0.25rem
    padding-bottom 0.25rem
    border-left none
    &.active
      font-weight 500
      opacity: 1 !important;
</style>
