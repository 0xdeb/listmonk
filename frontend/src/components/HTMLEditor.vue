<template>
  <div ref="htmlEditor" id="html-editor" class="html-editor" />
</template>

<script>
import CodeFlask from 'codeflask';
import { colors } from '../constants';

export default {
  props: {
    value: { type: String, default: '' },
    language: { type: String, default: 'html' },
    disabled: Boolean,
  },

  data() {
    return {
      data: '',
      flask: null,
    };
  },

  methods: {
    initHTMLEditor(body) {
      // CodeFlask editor is rendered in a shadow DOM tree to keep its styles
      // sandboxed away from the global styles.
      const el = document.createElement('code-flask');
      el.attachShadow({ mode: 'open' });
      el.shadowRoot.innerHTML = `
        <style>
          .codeflask .codeflask__flatten { 
            font-size: 15px;
            white-space: pre-wrap ;
            word-break: break-word ;
          }
          .codeflask .codeflask__lines { background: #fafafa; z-index: 10; }
          .codeflask .token.tag { font-weight: bold; }
          .codeflask .token.attr-name { color: #111; }
          .codeflask .token.attr-value { color: ${colors.primary} !important; }
        </style>
        <div id="area"></area>
      `;
      this.$refs.htmlEditor.appendChild(el);

      this.flask = new CodeFlask(el.shadowRoot.getElementById('area'), {
        language: this.$props.language,
        lineNumbers: false,
        styleParent: el.shadowRoot,
        readonly: this.disabled,
      });

      this.flask.onUpdate((v) => {
        this.data = v;
        this.$emit('input', v);
      });

      // Set the initial value.
      this.flask.updateCode(body);

      this.$nextTick(() => {
        document.querySelector('code-flask').shadowRoot.querySelector('textarea').focus();
      });
    },
  },

  mounted() {
    this.initHTMLEditor(this.$props.value || '');
  },

  watch: {
    value(newVal) {
      if (newVal !== this.data) {
        this.flask.updateCode(newVal);
      }
    },
  },
};
</script>
