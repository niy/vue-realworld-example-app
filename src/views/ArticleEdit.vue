<template>
  <div class="editor-page">
    <div class="container page">
      <div class="row">
        <div class="col-md-10 offset-md-1 col-xs-12">
          <RwvListErrors :errors="errors"/>
          <form v-on:submit.prevent="onPublish(article.slug, article)">
            <fieldset :disabled="inProgress">
              <fieldset class="form-group">
                <input
                  type="text"
                  class="form-control form-control-lg"
                  v-model="article.title"
                  placeholder="Article Title">
              </fieldset>
              <fieldset class="form-group">
                <input
                  type="text"
                  class="form-control"
                  v-model="article.description"
                  placeholder="What's this article about?">
              </fieldset>
              <fieldset class="form-group">
                <textarea
                  class="form-control"
                  rows="8"
                  v-model="article.body"
                  placeholder="Write your article (in markdown)">
                </textarea>
              </fieldset>
              <fieldset class="form-group">
                <div class="video-thumb" v-if="article.video && article.video.thumbnail_url">
                  <img class="img-thumbnail" v-bind:src="article.video.thumbnail_url"/>
                  <button class="btn btn-danger" @click="removeVideo(article)" v-bind:disabled="inProgress">x</button>
                </div>
                <div class="uploadBox">
                  <input type="file" accept="video/mp4, video/avi" @change="onFileSelect($event.target.files, article)"/>
                  <p v-if="!videoFile">Click to select a video or drop your file here...</p>
                  <p v-if="videoFile && !inProgress">{{ videoFile.name }}</p>
                  <p v-if="videoFile && inProgress">Uploading...</p>
                </div>
              </fieldset>
              <fieldset class="form-group">
                <input
                  type="text"
                  class="form-control"
                  placeholder="Enter tags"
                  v-model="tagInput"
                  v-on:keypress.enter.prevent="addTag(tagInput)">
                <div class="tag-list">
                  <span
                    class="tag-default tag-pill"
                    v-for="(tag, index) of article.tagList"
                    :key="tag + index">
                  <i
                    class="ion-close-round"
                    v-on:click="removeTag(tag)">
                </i>
                {{ tag }}
              </span>
                </div>
              </fieldset>
            </fieldset>
            <button
              :disabled="inProgress"
              class="btn btn-lg pull-xs-right btn-primary"
              type="submit">
              Publish Article
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>
<style>
  .uploadBox input[type="file"] {
    width: 100%;
    height: 100px;
    opacity: 0;
    position: absolute;
    left: 0;
    top: 0;
  }

  .uploadBox {
    position: relative;
    overflow: hidden;
    height: 100px;
    text-align: center;
    border: 2px dashed #e3e3e3;
    font-size: 1.5em;
    color: #999;
    padding: 30px 10px;
  }
</style>
<script>
  import { mapGetters } from 'vuex'
  import store from '@/store'
  import RwvListErrors from '@/components/ListErrors'
  import {
    ARTICLE_PUBLISH,
    ARTICLE_EDIT,
    FETCH_ARTICLE,
    ARTICLE_EDIT_ADD_TAG,
    ARTICLE_EDIT_REMOVE_TAG,
    ARTICLE_RESET_STATE,
    ARTICLE_POST_VIDEO,
    ARTICLE_DELETE_VIDEO
  } from '@/store/actions.type'

  export default {
    name: 'RwvArticleEdit',
    components: { RwvListErrors },
    props: {
      previousArticle: {
        type: Object,
        required: false
      }
    },
    async beforeRouteUpdate (to, from, next) {
      // Reset state if user goes from /editor/:id to /editor
      // The component is not recreated so we use to hook to reset the state.
      await store.dispatch(ARTICLE_RESET_STATE)
      return next()
    },
    async beforeRouteEnter (to, from, next) {
      // SO: https://github.com/vuejs/vue-router/issues/1034
      // If we arrive directly to this url, we need to fetch the article
      await store.dispatch(ARTICLE_RESET_STATE)
      if (to.params.slug !== undefined) {
        await store.dispatch(FETCH_ARTICLE,
          to.params.slug,
          to.params.previousArticle
        )
      }
      return next()
    },
    async beforeRouteLeave (to, from, next) {
      await store.dispatch(ARTICLE_RESET_STATE)
      next()
    },
    data () {
      return {
        tagInput: null,
        inProgress: false,
        errors: {},
        videoFile: null
      }
    },
    computed: {
      ...mapGetters([
        'article'
      ])
    },
    methods: {
      onPublishSuccess (data) {
        this.inProgress = false
        this.$router.push({
          name: 'article',
          params: { slug: data.article.slug }
        })
      },
      onPublishError (response) {
        this.inProgress = false
        if (response.data) {
          this.errors = response.data.errors
        }
      },
      onPublish (slug, article) {
        let action = slug ? ARTICLE_EDIT : ARTICLE_PUBLISH
        this.inProgress = true
        this.$store
          .dispatch(action)
          .then(({ data }) => {
            if (this.videoFile !== null) {
              this.$store
                .dispatch(ARTICLE_POST_VIDEO, { slug: data.article.slug, videoFile: this.videoFile })
                .then(this.onPublishSuccess)
                .catch(this.onPublishError)
            } else {
              this.onPublishSuccess(data)
            }
          })
          .catch(this.onPublishError)
      },
      onFileSelect (files, article) {
        if (files.length) {
          this.videoFile = files[0]
        }
      },
      removeVideo (article) {
        this.$store.dispatch(ARTICLE_DELETE_VIDEO, article.slug)
      },
      removeTag (tag) {
        this.$store.dispatch(ARTICLE_EDIT_REMOVE_TAG, tag)
      },
      addTag (tag) {
        this.$store.dispatch(ARTICLE_EDIT_ADD_TAG, tag)
        this.tagInput = null
      }
    }
  }
</script>
