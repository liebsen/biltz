<template>
  <div class="container is-widescreen">
    <div class="content column fadeIn">
      <h3>
        <span class="icon">
          <span class="mdi mdi-layer-group"></span>
        </span>
        <span>Grupos</span>
      </h3>
      <form @submit.prevent="submit">
        <div class="field has-addons">
          <div class="control">
            <input ref="input" @keyup="inputTrigger" v-model="query" class="input is-rounded is-success" type="text" :placeholder="'name' | t" autofocus>
          </div>
          <div class="control">
            <button v-show="query.length" type="button" @click="clear" class="button is-rounded is-danger">
              <span class="icon">
                <span class="mdi mdi-close"></span>
              </span>
            </button>
            <button v-show="!query.length" type="submit" id="searchbtn" class="button is-rounded is-success">
              <span class="icon">
                <span class="mdi mdi-magnify"></span>
              </span>
            </button>
          </div>
        </div>
      </form>
      <div v-if="Object.keys(data).length" class="has-text-left">
        <table class="table">
          <thead>
            <th></th>
            <th>Nombre</th>
            <th>Rondas</th>
            <th>Minutos</th>
            <th>Compensación</th>
          </thead>
          <tbody>
            <tr v-for="(item, index) in data.data" :key="index">
              <td>
                <router-link :to="'/group/'+item._id">
                  <span class="icon">
                    <span class="mdi mdi-layers"></span>
                  </span>
                </router-link>
              </td>
              <td>
                <span v-html="item.code"></span>
              </td>
              <td>
                <span v-html="item.games"></span>
              </td>
              <td>
                <span v-html="item.minutes"></span>'
              </td>
              <td>
                +<span v-html="item.compensation"></span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <nav class="pagination is-centered is-rounded" role="navigation" aria-label="pagination">
      <!--a class="pagination-previous">Previous</a>
      <a class="pagination-next">Next page</a-->
      <ul class="pagination-list">
        <li v-for="(page, index) in pages" :key="index">
          <router-link :to="'?q=' + query + '&offset=' + page" class="pagination-link" :class="{'is-current': offset == page}" :title="'Ir a página ' + parseInt(page / limit + 1)"></router-link>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import axios from 'axios'
import snackbar from '../components/Snackbar'
export default {
  name: 'groups',
  watch: {
    '$route': function () {
      this.triggerSearch()
    }
  },
  mounted: function () {
    this.triggerSearch()
  },
  methods: {
    inputTrigger: function () {
      if (this.interval) clearInterval(this.interval)
      this.interval = setTimeout(() => {
        this.$router.push({ path: 'groups', query: { q: this.query } })
      }, 1500)
    },
    clear: function () {
      this.query = ''
      this.submit()
    },
    triggerSearch: function () {
      if (this.$route.query.q) {
        this.query = this.$route.query.q
      }
      if (this.$route.query.offset) {
        this.offset = parseInt(this.$route.query.offset)
      }
      this.$nextTick(() => this.$refs.input.focus())
      this.search()
    },
    search: function () {
      this.$root.loading = true
      axios.post('/groups', { query: this.query, offset: this.offset, limit: this.limit }).then((res) => {
        this.data = res.data

        var pages = []
        if (res.data.error) {
          if (res.data.error === 'not_enough_params') {
            snackbar('info', 'Ingresá una palabra clave para ver grupos.', 15000)
          }
        } else {
          if (res.data.count === 0) {
            snackbar('warning', 'No hay grupos', 5000)
          } else {
            var numPages = Math.ceil(res.data.count / this.limit)
            for (var i = 0; i < numPages; i++) {
              pages[i] = i * this.limit
            }
          }
        }

        let max = 20

        if (pages.length > max) {
          pages.splice(max / 2, pages.length - max)
        }

        this.pages = pages
        this.$root.loading = false
      })
    },
    submit: function () {
      this.$router.push('/groups?q=' + this.query.trim())
    }
  },
  data () {
    return {
      data: {},
      pages: {},
      query: '',
      limit: 10,
      offset: 0
    }
  }
}
</script>
