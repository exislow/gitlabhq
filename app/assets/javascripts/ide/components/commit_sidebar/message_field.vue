<script>
import { __, sprintf } from '../../../locale';
import Icon from '../../../vue_shared/components/icon.vue';
import popover from '../../../vue_shared/directives/popover';
import { MAX_TITLE_LENGTH, MAX_BODY_LENGTH } from '../../constants';

export default {
  directives: {
    popover,
  },
  components: {
    Icon,
  },
  props: {
    text: {
      type: String,
      required: true,
    },
    placeholder: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      scrollTop: 0,
      isFocused: false,
    };
  },
  computed: {
    allLines() {
      return this.text.split('\n').map((line, i) => ({
        text: line.substr(0, this.getLineLength(i)) || ' ',
        highlightedText: line.substr(this.getLineLength(i)),
      }));
    },
  },
  methods: {
    handleScroll() {
      if (this.$refs.textarea) {
        this.$nextTick(() => {
          this.scrollTop = this.$refs.textarea.scrollTop;
        });
      }
    },
    getLineLength(i) {
      return i === 0 ? MAX_TITLE_LENGTH : MAX_BODY_LENGTH;
    },
    onInput(e) {
      this.$emit('input', e.target.value);
    },
    updateIsFocused(isFocused) {
      this.isFocused = isFocused;
    },
  },
  popoverOptions: {
    trigger: 'hover',
    placement: 'top',
    content: sprintf(
      __(`
        The character highlighter helps you keep the subject line to %{titleLength} characters
        and wrap the body at %{bodyLength} so they are readable in git.
      `),
      { titleLength: MAX_TITLE_LENGTH, bodyLength: MAX_BODY_LENGTH },
    ),
  },
};
</script>

<template>
  <fieldset class="common-note-form ide-commit-message-field">
    <div
      :class="{
        'is-focused': isFocused,
      }"
      class="md-area"
    >
      <div v-once class="md-header">
        <ul class="nav-links">
          <li>
            {{ __('Commit Message') }}
            <span v-popover="$options.popoverOptions" class="form-text text-muted prepend-left-10">
              <icon name="question" />
            </span>
          </li>
        </ul>
      </div>
      <div class="ide-commit-message-textarea-container">
        <div class="ide-commit-message-highlights-container">
          <div
            :style="{
              transform: `translate3d(0, ${-scrollTop}px, 0)`,
            }"
            class="note-textarea highlights monospace"
          >
            <div v-for="(line, index) in allLines" :key="index">
              <span v-text="line.text"> </span
              ><mark v-show="line.highlightedText" v-text="line.highlightedText"> </mark>
            </div>
          </div>
        </div>
        <textarea
          ref="textarea"
          :placeholder="placeholder"
          :value="text"
          class="note-textarea ide-commit-message-textarea"
          name="commit-message"
          @scroll="handleScroll"
          @input="onInput"
          @focus="updateIsFocused(true);"
          @blur="updateIsFocused(false);"
        >
        </textarea>
      </div>
    </div>
  </fieldset>
</template>
