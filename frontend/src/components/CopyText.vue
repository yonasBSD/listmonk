<template>
  <a href="#" class="copy-text" ref="text" @click.prevent="onClick">
    <template v-if="!hideText">{{ $props.text }}</template>
    <b-icon icon="file-multiple-outline" size="is-small" />
  </a>
</template>

<script>
export default {
  name: 'CopyText',

  props: {
    text: { type: String, default: '' },
    hideText: { type: Boolean, default: false },
  },

  methods: {
    onClick(e) {
      e.preventDefault();
      e.stopPropagation();

      const input = document.createElement('input');
      input.setAttribute('type', 'text');
      input.style.opacity = '0';
      input.value = this.$props.text;
      document.body.appendChild(input);
      input.select();
      document.execCommand('copy');
      document.body.removeChild(input);

      this.$utils.toast(this.$t('globals.messages.copied'));
    },
  },
};
</script>
