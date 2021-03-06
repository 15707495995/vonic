<template>
  <transition name="nav-fade">
    <div von-nav v-show="!hideNavbar" class="navbar" no-transtion>
      <transition name="nav-item-fade">
        <div class="back-button"
          v-if="showBackButton"
          v-html="backButtonText"
          @click="backButtonClicked($event)"
        >
        </div>
      </transition>

      <transition name="nav-item-fade">
        <div class="menu-button"
          v-if="showMenuButton"
          v-html="menuButtonText"
          @click="menuButtonClicked($event)"
        >
        </div>
      </transition>
    </div>
  </transition>
</template>
<style lang='scss'>
  @import "../scss/variables";
  @import '../scss/mixins';
  $themeColor: '#007aff';

  $navbar-z-index: 10;
  $navbar-title-z-index: 12;
  $navbar-button-z-index: 13;

  .navbar.visible {
    visibility: visible;
  }

  .navbar {
    visibility: hidden;

    box-sizing: border-box;
    -webkit-tap-highlight-color: rgba(0,0,0,0);
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 44px;
    z-index: $navbar-z-index;
    background-color: #fff;

    /* 用阴影替代 */
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.15);

    /*
    &:after {
      @include hairline(bottom);
    }
    */

    .back-button, .menu-button {
      position: absolute;
      top: 0;
      width: 80px;
      height: 44px;
      line-height: 44px;
      z-index: $navbar-button-z-index;

      .button-icon {
        padding: 0;
        min-height: 44px;
        height: 44px;
        margin-top: -1px;

        &.ion-android-arrow-back {
          &:before {
            font-size: 28px;
          }
        }
      }
    }

    .back-button {
      left: 0;
      padding: 0 0 0 10px;
      text-align: left;
    }

    .menu-button {
      right: 0;
      padding: 0 10px 0 0;
      text-align: right;
    }

    .center {
      position: absolute;
      top: 0px;
      left: 0;
      width: 100%;
      height: 44px;
      padding: 0;
      text-align: center;
      z-index: $navbar-title-z-index;

      .title {
        display: inline-block;
        font-size: 18px;
        line-height: 44px;

        &.title-transition {
          @include transition-duration($ios-transition-duration);
          @include transition-timing-function($ios-transition-timing-function);
          -webkit-transition-property: opacity, -webkit-transform, box-shadow;
          transition-property: opacity, transform, box-shadow;
        }
      }
    }

    .nav-item-fade-enter-active,
    .nav-item-fade-leave-active {
      @include transition-duration($ios-transition-duration);
      @include transition-timing-function($ios-transition-timing-function);
    }

    .nav-item-fade-enter,
    .nav-item-fade-leave-to {
      opacity: 0;
    }

    .nav-item-fade-leave,
    .nav-item-fade-enter-to {
      opacity: 1;
    }
  }

  /* android or other */
  .grade-b .navbar {
    .center .title,
    .fade-transition
    {
      @include transition-property(all);
      @include transition-duration($android-transition-duration);
      @include transition-timing-function($android-transition-timing-function);
    }
  }

  .nav-fade-enter-active,
  .nav-fade-leave-active
  {
    @include transition-duration($ios-transition-duration);
    @include transition-timing-function($ios-transition-timing-function);
  }

  .nav-fade-enter,
  .nav-fade-leave-to {
    opacity: 0;
  }

  .nav-fade-leave,
  .nav-fade-enter-to {
    opacity: 1;
  }

  [no-transition] {
    @include transition-duration(0ms !important);
  }

</style>
<script>
  import channel from './channel'

  const is_ios_device = () => {
    return /iPad|iPhone|iPod/.test(navigator.userAgent)
  }

  function getTitleTransitionDistance(t) {
    // return (document.body.offsetWidth - t.offsetWidth) / 2 - 10
    return (document.body.offsetWidth - t.offsetWidth) / 2
  }

  function centerElement(navbar, title, direction) {
    let centerId = Math.random().toString(36).substr(3, 6)
    let c = document.createElement('div')
    c.id = centerId
    c.className = 'center'
    let t = document.createElement('span')

    if (!window.__disable_nav_title_transition__) {
      t.className = 'title title-transition'
    } else {
      t.className = 'title'
    }

    t.innerHTML = title

    let reverse = direction == 'back'
    t.style.opacity = 0

    if (!window.__disable_nav_title_transition__ && is_ios_device()) {
      t.style.transform = 'translate3d(' + (reverse ? '-' : '') + getTitleTransitionDistance(t) + 'px,0,0)'
      t.style.webkitTransform = 'translate3d(' + (reverse ? '-' : '') + getTitleTransitionDistance(t) + 'px,0,0)'
    }

    if (!navbar.querySelector('.center')) {
      t.style.opacity = 1
      t.style.transform = 'translate3d(0,0,0)'
      t.style.webkitTransform = 'translate3d(0,0,0)'
    }

    c.appendChild(t)
    navbar.appendChild(c)
    return document.getElementById(centerId)
  }

  function titleIn(t) {
    t.style.opacity = 1
    t.style.transform = 'translate3d(0,0,0)'
    t.style.webkitTransform = 'translate3d(0,0,0)'
  }

  function titleOut(t, direction) {
    let reverse = direction == 'back'
    t.style.opacity = 0
    if (!window.__disable_nav_title_transition__ && is_ios_device()) {
      t.style.transform = 'translate3d(' + (reverse ? '' : '-') + getTitleTransitionDistance(t) + 'px,0,0)'
      t.style.webkitTransform = 'translate3d(' + (reverse ? '' : '-') + getTitleTransitionDistance(t) + 'px,0,0)'
    }
  }

  function defaultBackButtonText() {
    return is_ios_device() ?
      '<a class="button button-icon icon ion-ios-arrow-back"></a>' :
      '<a class="button button-icon icon ion-android-arrow-back"></a>'
  }

  function defaultMenuButtonText() {
    return '<a class="button button-icon icon ion-navicon"></a>'
  }

  window.__block_touch__ = false

  function navTransitionStart() {
    var navbar = document.querySelector('.navbar')
    navbar.style.position = 'absolute'
    window.__block_touch__ = true
  }

  function navTransitionEnd() {
    var navbar = document.querySelector('.navbar')
    navbar.style.position = 'fixed'
    window.__block_touch__ = false
  }

  let is_first_render = true

  export default {
    data() {
      return {
        title: '',
        showBackButton: false,
        onBackButtonClick: undefined,
        showMenuButton: false,
        onMenuButtonClick: undefined,
        backButtonText: defaultBackButtonText(),
        menuButtonText: defaultMenuButtonText(),
        hideNavbar: false
      }
    },

    created() {
      // center & center leave
      let c, cl;

      channel.$on('PageTransitionEvent', (data) => {
        if (is_first_render) {
          function setNavbarVisible() {
            let nav = document.querySelector('[von-nav]')
            nav.classList.add('visible')
            nav.removeAttribute('no-transition')
          }
          if (!!data.hideNavbar) {
            setTimeout(setNavbarVisible, is_ios_device() ? 500 : 200)
          } else {
            setNavbarVisible()
          }
          is_first_render = false
        }

        let direction = document.querySelector('[von-app]').getAttribute('transition-direction')
        this.title = data.title ? data.title : ''
        this.hideNavbar = !!data.hideNavbar

        // 浏览器前进、后退按钮被连续点击
        if (document.querySelectorAll('[von-nav] .center').length > 1) {
          c.querySelector('.title').innerHTML = this.title
          return
        }

        navTransitionStart()

        c = centerElement(this.$el, this.title, direction)

        setTimeout(() => {
          titleIn(c.querySelector('.title'))
          if (cl) {
            titleOut(cl.querySelector('.title'), direction)
          }

          setTimeout(() => {
            if (cl) this.$el.removeChild(cl)
            cl = c
            navTransitionEnd()
          }, window.__disable_nav_title_transition__ ? 0 : 600)
        })

        // nav item & click event handler
        this.showBackButton = data.showBackButton
        this.onBackButtonClick = data.onBackButtonClick
        if (data.backButtonText)
          this.backButtonText = data.backButtonText
        else
          this.backButtonText = defaultBackButtonText()

        this.showMenuButton = data.showMenuButton
        this.onMenuButtonClick = data.onMenuButtonClick
        if (data.menuButtonText)
          this.menuButtonText = data.menuButtonText
        else
          this.menuButtonText = defaultMenuButtonText()
      })
    },

    methods: {
      backButtonClicked(e) {
        if (window.__block_touch__) {
          e.preventDefault()
          return;
        }

        if (this.onBackButtonClick) {
          this.onBackButtonClick()
          return
        }

        document.querySelector('[von-app]').setAttribute('transition-direction', 'back')
        history.go(-1)
      },

      menuButtonClicked() {
        if (window.__block_touch__) {
          e.preventDefault()
          return;
        }

        if (this.onMenuButtonClick) {
          this.onMenuButtonClick()
        }
      }
    }
  }

</script>
