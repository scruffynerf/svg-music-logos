<template>
  <header class="header">
    <div class="header__main">
      <h1 class="header__title">
        <a href="http://tiagoporto.github.io/svg-music-logos" class="header__link-title">
          <!-- <img id="logo" src="img/logos/logo.svg" alt="{-}" width="250" /> -->
          SVG Music Logos
        </a>
        <span class="header__subtitle">{{artists.length}} artists • {{logos.length}} logos</span>
      </h1>

      <div class="github-buttons">
        <a class="github-button" href="https://github.com/tiagoporto" data-show-count="true" aria-label="Follow @tiagoporto on GitHub">Follow @tiagoporto</a>

        <a class="github-button" href="https://github.com/tiagoporto/svg-music-logos" data-icon="octicon-star" data-show-count="true" aria-label="Star tiagoporto/svg-music-logos on GitHub">Star</a>
      </div>

      <p>A collection of bands' and musicians' logos in SVG.</p>
      <p>All brands are trademarks of their respective bands or musicians.</p>
      <p>The brands and symbols should only be used to represent which artists they refer.</p>

      <p><a href="https://github.com/tiagoporto/svg-music-logos/issues" class="link">Request a new logo or report a problem.</a></p>

      <form>
        <input type="search" v-model.trim="search.artist" placeholder="Search" class="search" autofocus/>

        <button type="button">
          <svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32' >
            <path d='M16,21.625c2.682,0,4.875-2.25,4.875-5V6.375c0-2.75-2.193-5-4.875-5c-2.682,0-4.875,2.25-4.875,5v10.25  C11.125,19.375,13.318,21.625,16,21.625z M21.876,11.5v5.125c0,3.309-2.636,6-5.876,6s-5.875-2.691-5.875-6V11.5H7.126v5.125  c0,4.443,3.194,8.132,7.372,8.861v2.139h-3.372v3h9.749v-3h-3.376v-2.139c4.181-0.729,7.375-4.418,7.375-8.861V11.5H21.876z' />
          </svg>
        </button>
      </form>

      <div class="filter">
        <select v-model="search.origin" class="select">
            <option value="">All Origins</option>
            <option v-for="origin in origins" :value="origin">{{origin}}</option>
        </select>

        <select v-model="search.genre" class="select">
            <option value="">All Genres</option>
            <option v-for="genre in genres" :value="genre">{{genre}}</option>
        </select>
      </div>
    </div>
  </header>
</template>

<script>
import './AppHeader.styl'
import './Jumbotron.styl'
import _ from 'lodash'
import annyang from 'annyang'

const setJumbotronHeight = () => {
  if (window.innerWidth > 768) {
    if (window.scrollY > 20) {
      document.getElementById('jumbotron').style.height = '100%'
    } else {
      document.getElementById('jumbotron').style.height = '400px'
    }
  } else {
    document.getElementById('jumbotron').style.height = '100%'
  }
}

window.addEventListener('scroll', _.debounce(setJumbotronHeight, 20))

export default {
  name: 'AppHeader',
  props: {
    artists: [Array],
    origins: [Array],
    genres: [Array],
    search: [Object],
    logos: [Array]
  },
  mounted () {
    const commands = {
      'search *artist': param => {
        this.search.artist = param
      },
      'clean search': () => {
        this.search.artist = ''
        this.search.origin = ''
        this.search.genre = ''
      },
      'filter by *filter': filter => {
        const param = filter.replace(/(?:^|\s)\s/g, a => a.touppercase())

        if (this.genres.includes(param)) {
          this.search.genre = param
        } else if (this.origins.includes(param)) {
          this.search.origin = param
        }
      },
      'clean filters': () => {
        this.search.origin = ''
        this.search.genre = ''
      }
    }

    annyang.addCommands(commands)

    annyang.start()
  }
}

</script>
