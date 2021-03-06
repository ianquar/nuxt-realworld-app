<template>
  <div class="editor-page">
    <div class="container page">
      <div class="row">

        <div class="col-xs-12 col-md-10 offset-md-1">
          <error-messages v-show="Object.keys(error).length"
                          :error="error" />
          <form @submit.prevent="crateArticle">
            <fieldset>
              <fieldset class="form-group">
                <input type="text"
                       class="form-control form-control-lg"
                       placeholder="Article Title"
                       v-model="title"
                       :disabled="submitting">
              </fieldset>
              <fieldset class="form-group">
                <input type="text"
                       class="form-control"
                       placeholder="What's this article about?"
                       v-model="description"
                       :disabled="submitting">
              </fieldset>
              <fieldset class="form-group">
                <textarea class="form-control"
                          rows="8"
                          placeholder="Write your article (in markdown)"
                          v-model="body"
                          :disabled="submitting"></textarea>
              </fieldset>
              <fieldset class="form-group">
                <input type="text"
                       class="form-control"
                       placeholder="Enter tags"
                       v-model="tagInput"
                       :disabled="submitting"
                       @keypress.enter.prevent="addTag(tagInput)">
                <div class="tag-list">
                  <span v-for="(n, idx) in tagList"
                        :key="n + idx"
                        class="tag-default tag-pill">
                    <i class="ion-close-round"
                       @click="removeTag(n)"></i>
                    {{ n }}
                  </span>
                </div>
              </fieldset>
              <button class="btn btn-lg pull-xs-right btn-primary"
                      :disabled="submitting">
                Publish Article
              </button>
            </fieldset>
          </form>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import ErrorMessages from '@/components/ErrorMessages'

export default {
  name: 'editor',
  components: {
    ErrorMessages
  },
  middleware: 'auth-required',
  asyncData ({ params, store, redirect }) {
    let data = {
      title: '',
      description: '',
      body: '',
      tagList: [],
      tagInput: '',
      submitting: false,
      error: {}
    }

    return params.slug
      ? store.dispatch('api/request', {
        promise: store.dispatch('api/getArticle', { slug: params.slug }),
        success (res) {
          let { author, title, description, body, tagList } = res.data.article
          let currentUsername = store.state.auth.user.username

          return author.username === currentUsername
            ? {
              ...data,
              title,
              description,
              body,
              tagList
            }
            : redirect({ name: 'index' })
        }
      })
      : data
  },
  head () {
    return {
      title: 'Editor - Conduit'
    }
  },
  methods: {
    addTag (tag) {
      if (this.tagList.indexOf(tag) === -1) {
        this.tagList.push(tag)
        this.tagInput = ''
      }
    },
    removeTag (tag) {
      let idx = this.tagList.findIndex(tagElm => tagElm === tag)
      this.tagList.splice(idx, 1)
    },
    crateArticle () {
      let actionName = this.$route.params.slug
        ? 'api/updateArticle'
        : 'api/createArticle'

      this.submitting = true
      this.$store.dispatch('api/request', {
        promise: this.$store.dispatch(actionName, {
          slug: this.$route.params.slug,
          data: {
            article: {
              title: this.title,
              description: this.description,
              body: this.body,
              tagList: this.tagList
            }
          }
        }),
        success: (res) => {
          this.$router.push({
            name: 'article-slug',
            params: {
              slug: res.data.article.slug
            }
          })
        },
        fail: (err) => {
          this.submitting = false
          this.error = err.response.data.errors
        }
      })
    }
  }
}
</script>

