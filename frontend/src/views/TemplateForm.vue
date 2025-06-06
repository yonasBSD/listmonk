<template>
  <section>
    <form @submit.prevent="onSubmit">
      <div class="modal-card content template-modal-content" style="width: auto">
        <header class="modal-card-head">
          <b-button @click="onTogglePreview" class="is-pulled-right" type="is-primary" icon-left="file-find-outline">
            {{ $t('templates.preview') }} (F9)
          </b-button>

          <template v-if="isEditing">
            <h4>{{ data.name }}</h4>
            <p class="has-text-grey is-size-7">
              {{ $t('globals.fields.id') }}: <span data-cy="id"><copy-text :text="`${data.id}`" /></span>
            </p>
          </template>
          <h4 v-else>
            {{ $t('templates.newTemplate') }}
          </h4>
        </header>
        <section expanded class="modal-card-body mb-0 pb-0">
          <div class="columns">
            <div class="column is-9">
              <b-field :label="$t('globals.fields.name')" label-position="on-border">
                <b-input :maxlength="200" :ref="'focus'" v-model="form.name" name="name"
                  :placeholder="$t('globals.fields.name')" required />
              </b-field>
            </div>
            <div class="column is-3">
              <b-field :label="$t('globals.fields.type')" label-position="on-border">
                <b-select v-model="form.type" :disabled="isEditing" expanded>
                  <option value="campaign">
                    {{ $tc('templates.typeCampaignHTML') }}
                  </option>
                  <option value="campaign_visual">
                    {{ $tc('templates.typeCampaignVisual') }}
                  </option>
                  <option value="tx">
                    {{ $tc('templates.typeTransactional') }}
                  </option>
                </b-select>
              </b-field>
            </div>
          </div>
          <div class="columns" v-if="form.type === 'tx'">
            <div class="column is-12">
              <b-field :label="$t('templates.subject')" label-position="on-border">
                <b-input :maxlength="200" :ref="'focus'" v-model="form.subject" name="name"
                  :placeholder="$t('templates.subject')" required />
              </b-field>
            </div>
          </div>

          <template v-if="form.body !== null">
            <b-field v-if="form.type === 'campaign_visual'" label-position="on-border" class="mb-1">
              <visual-editor v-if="form.type === 'campaign_visual'" name="body" :source="form.bodySource"
                @change="onChangeVisualEditor" height="70vh" />
            </b-field>

            <b-field v-else :label="$t('templates.rawHTML')" label-position="on-border">
              <code-editor lang="html" v-model="form.body" name="body" />
            </b-field>
          </template>

          <p class="is-size-7">
            <template v-if="form.type === 'campaign'">
              {{ $t('templates.placeholderHelp', { placeholder: egPlaceholder }) }}
            </template>
            <a target="_blank" rel="noopener noreferer" href="https://listmonk.app/docs/templating">
              {{ $t('globals.buttons.learnMore') }}
            </a>
          </p>
        </section>
        <footer class="modal-card-foot has-text-right">
          <b-button @click="$parent.close()">
            {{ $t('globals.buttons.close') }}
          </b-button>
          <b-button v-if="$can('templates:manage')" native-type="submit" type="is-primary" :loading="loading.templates">
            {{ $t('globals.buttons.save') }}
          </b-button>
        </footer>
      </div>
    </form>
    <campaign-preview v-if="previewItem" is-post type="template" :title="previewItem.name"
      :template-type="previewItem.type" :body="form.body" @close="onTogglePreview" />
  </section>
</template>

<script>
import Vue from 'vue';
import { mapState } from 'vuex';
import CampaignPreview from '../components/CampaignPreview.vue';
import CodeEditor from '../components/CodeEditor.vue';
import VisualEditor from '../components/VisualEditor.vue';
import CopyText from '../components/CopyText.vue';

export default Vue.extend({
  components: {
    CampaignPreview,
    CopyText,
    'code-editor': CodeEditor,
    'visual-editor': VisualEditor,
  },

  props: {
    data: { type: Object, default: () => { } },
    isEditing: { type: Boolean, default: false },
  },

  data() {
    return {
      // Binds form input values.
      form: {
        name: '',
        subject: '',
        type: 'campaign',
        optin: '',
        body: null,
        bodySource: null,
      },
      previewItem: null,
      egPlaceholder: '{{ template "content" . }}',
    };
  },

  methods: {
    onTogglePreview() {
      this.previewItem = !this.previewItem ? this.form : null;
    },

    onPreviewShortcut(e) {
      if (e.key === 'F9') {
        this.onTogglePreview();
        e.preventDefault();
      }
    },

    onSubmit() {
      if (this.isEditing) {
        this.updateTemplate();
        return;
      }

      this.createTemplate();
    },

    createTemplate() {
      const data = {
        id: this.data.id,
        name: this.form.name,
        type: this.form.type,
        subject: this.form.subject,
        body: this.form.body,
        body_source: this.form.bodySource,
      };

      this.$api.createTemplate(data).then((d) => {
        this.$emit('finished');
        this.$parent.close();
        this.$utils.toast(this.$t('globals.messages.created', { name: d.name }));
      });
    },

    updateTemplate() {
      const data = {
        id: this.data.id,
        name: this.form.name,
        type: this.form.type,
        subject: this.form.subject,
        body: this.form.body,
        body_source: this.form.bodySource,
      };

      this.$api.updateTemplate(data).then((d) => {
        this.$emit('finished');
        this.$parent.close();
        this.$utils.toast(`'${d.name}' updated`);
      });
    },

    onChangeVisualEditor({ source, body }) {
      this.form.body = body;
      this.form.bodySource = source;
    },
  },

  computed: {
    ...mapState(['loading']),
  },

  mounted() {
    this.form = { ...this.$props.data };

    this.$nextTick(() => {
      this.$refs.focus.focus();
    });

    window.addEventListener('keydown', this.onPreviewShortcut);
  },

  beforeDestroy() {
    window.removeEventListener('keydown', this.onPreviewShortcut);
  },
});
</script>
