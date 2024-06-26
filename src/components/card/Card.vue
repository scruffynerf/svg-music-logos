<template>
  <div class="card" :class="{ 'card--inverse': band.logo.inverse }">
    <logo :band="band"></logo>

    <div class="card__content">
      <h2
        v-if="band.nameTemplate"
        class="card__title"
        v-html="band.nameTemplate"
      ></h2>

      <h2 v-else class="card__title">
        <a
          class="card__link"
          :href="band.link"
          :title="`${band.name}'s website`"
          >{{ band.name }}</a
        >
      </h2>

      <p v-if="band.genre">
        Genre:
        <template v-for="(genre, index) in band.genre">
          {{ genre }}
          <template v-if="index < band.genre.length - 1">•</template>
        </template>
      </p>

      <p>
        Origin:
        <template v-for="(origin, index) in band.origin">
          <flag :key="index" :iso="getFlagIso(origin)" :squared="false" />
          {{ origin }}
          <template v-if="index < band.origin.length - 1">/</template>
        </template>
      </p>
      <p>Reference: {{ band.logo.title }}</p>
    </div>

    <div class="card__footer">
      <button class="card__button" @click="download($event, band)">
        Download SVG
      </button>
    </div>
  </div>
</template>

<script>
import 'whatwg-fetch'
import './Card.styl'
import { deburr, kebabCase } from 'lodash'
import FileSaver from 'file-saver'
import FlagIso from './FlagIso.json'
import Logo from './Logo.vue'

export default {
  name: 'Card',
  components: {
    Logo,
  },
  props: {
    band: [Object],
  },
  methods: {
    save(content, filename) {
      content = new Blob([content], { type: 'text/plain' })
      FileSaver.saveAs(content, filename)

      if (process.env.NODE_ENV === 'production') {
        this.$ga.event({
          eventCategory: 'download',
          eventAction: 'click',
          eventLabel: filename,
        })
      }
    },
    download(event, band) {
      const bandName = kebabCase(deburr(band.name.toLowerCase()))
      const versionName = kebabCase(deburr(band.logo.title.toLowerCase()))
      const filename = `${bandName}_${versionName}.svg`
      const svgFileName = band.logo.svg

      fetch(`logos/${band.folder}/${svgFileName}`)
        .then((response) => response.text())
        .then((response) => {
          let svg = response

          if (band.logo.cls) {
            if (!svg.match(/<svg[\w\s\t\n:="\\'/.#-]+ class="(.*?)"/)) {
              svg = svg.replace(/(<svg[\w\s\t\n:="\\'/.#-]+)/, '$1 class=" "')
            }

            const svgClass =
              svg.match(/<svg[\w\s\t\n:="\\'/.#-]+ class="(.*?)"/) &&
              svg.match(/class="(.*?)"/)[1].split(' ')

            const classToAdd = band.logo.cls.split(' ')
            const allClasses = [...svgClass, ...classToAdd].filter(
              (classname) => classname,
            )
            const newClasses = [...new Set(allClasses)].join(' ')
            svg = svg.replace(
              /(<svg[\w\s\t\n:="\\'/.#-]+) class="[\w\s-_]+?"/,
              `$1 class="${newClasses}"`,
            )
          }

          if (band.css) {
            fetch(`logos/${band.folder}/${band.css}`)
              .then((response) => response.text())
              .then((response) => {
                const styles = `<style>\r\n${response}\r</style>`
                svg = svg.replace(/(<svg[\w='"\s:/.-]+>)/, `$1\r\n${styles}`)

                this.save(svg, filename)
              })
          } else {
            this.save(svg, filename)
          }
        })
    },
    getFlagIso(country) {
      return FlagIso[country]
    },
  },
}
</script>
