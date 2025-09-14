<script setup lang="ts">

import { ref } from 'vue'

import {renderEditor} from '@lblod/embeddable-say-editor';
import { useTemplateRef, onMounted } from 'vue'

// the first argument must match the ref value in the template
const editorElement = useTemplateRef('editor-element')


const pluginDemoConfig = {
  title: 'my editor',
  width: '100%',
  height: '800px',
  plugins: [
    'citation',
    'besluit',
    'besluit-topic',
    'lpdc',
    // 'article-structure',
    'variable',
    'table-of-contents',
    'roadsign-regulation',
    'formatting-toggle',
    'template-comments',
    'confidentiality',
    'html-edit',
    'html-preview',
    'location',
  ],
  options: {
    docContent: 'table_of_contents? block+',
    citation: {
      type: 'ranges',
      activeInRanges: (state) => [[0, state.doc.content.size]],
    },
    besluitTopic: {
      endpoint: 'https://data.vlaanderen.be/sparql',
      widgetLocation: 'sidebar',
    },
    lpdc: {
      endpoint:
        'https://embeddable.dev.gelinkt-notuleren.lblod.info/lpdc-service/doc/instantie',
    },
    besluit: {
      fullLengthArticles: false,
      onlyArticleSpecialName: true,
    },
    location: {
      locationTypes: ['address', 'place'],
    },
  },
};
onMounted(async () => {
  const editor =await renderEditor({element:editorElement.value, plugins:
  [...pluginDemoConfig.plugins, "rdfa-editor"], options:
  {...pluginDemoConfig.options, shadowDom: false}})
  editor.setHtmlContent('hello world')
})

</script>

<template>
  <div ref="editor-element" style="height: 800px; width: 1500px">
  </div>

</template>
