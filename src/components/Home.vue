<template>
  <v-container class="fill-height">
    <v-responsive
      class="align-centerfill-height mx-auto"
      max-width="100%"
    >
      <v-row>
        <v-col cols="6">
          <v-card width="100%" color='' variant="tonal" >
            <v-textarea class='pa-1 ma-1' rows="30" row-height="2rem" name="input-7-1" label="Place your CATP (Japanese) record here" v-model="text">
            </v-textarea>
          </v-card>
        </v-col>
        <v-col cols="6">
          <v-card width="100%" color='' variant="tonal">
            <v-card-actions class="justify-end" v-if="text.trim().length > 0" style='position: absolute;right:0px'>
              <v-tooltip text="Copy Text" location="bottom">
                <template v-slot:activator="{ props }">
                  <v-btn small density="compact" icon="mdi-content-copy" @click="doCopy" variant="text" color="rgb(236, 106, 106)" v-bind="props" ></v-btn>
                </template>
              </v-tooltip>
            </v-card-actions>          
            <v-card-actions>
              <v-card-text class="pa-0 text--primary text-left" @change="v => translatedText = v" >
                <label class="v-label v-label--active v-label--is-disabled theme--light" style="left: 0px; position: relative; display:inline"></label>
                <div id="translatedPre" style="min-height:6rem;overflow-y: auto;overflow-x: auto;font-size: 1em; text-wrap:nowrap;height:55em" class="text-left">
                  <div v-html="translatedHTML"></div>
                </div>
              </v-card-text>
            </v-card-actions>
            </v-card>
        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script setup>
  import { defineComponent, ref, watch } from 'vue'
  import useClipboard from 'vue-clipboard3'
  import {CAPTParse, PICAXRewrite} from 'crossasia-catp-converter'
  const { toClipboard } = useClipboard()
  const text = ref('')
  const translatedText = ref('')
  const translatedHTML = ref('')
  watch(text, () => {
    const catpRecords = parseTextToArray(text.value)
    const PICAXRecords = catpRecords.map(record =>  PICAXRewrite(CAPTParse(record)))
    translatedHTML.value = PICAXRecords.map(record => record.map(x => typeof x === 'string' ? x : x.toHTML(x)).join('<br/>')).join('<hr/>')
    translatedText.value = PICAXRecords.map(record => record.join('\n')).join('\n\n\n')
  })
  const doCopy = async () => {
    try {
      await toClipboard(translatedText.value)
    } catch (e) {
    }
  }
  function parseTextToArray(inputText) {
    // Split the text by line breaks
    const lines = inputText.split(/\r?\n/);
    
    const result = [];
    let tempArray = [];

    lines.forEach(line => {
        // If the line is not empty or contains only spaces, add it to the current group
        if (line.trim() !== "") {
            tempArray.push(line.trim());
        } else if (tempArray.length > 0) {
            // When encountering an empty line and tempArray has values, push the group to result
            result.push(tempArray);
            tempArray = [];
        }
    });

    // Push the last group if any exists (in case the input doesn't end with an empty line)
    if (tempArray.length > 0) {
        result.push(tempArray);
    }

    return result;
}
</script>
<style>
  #translatedPre span.code {
    margin-right: 1em;
    color:rgb(236, 106, 106)
  }
  #translatedPre span.subCode {
    color:rgb(119, 119, 245)
  }
  #translatedPre span.lang {
    color:rgb(80, 165, 80)
  }
  #translatedPre span.dollar {
    color:rgb(204, 134, 204)
  }
  #translatedPre hr {
    margin-top: 1em;
    margin-bottom: 1em;
  }
</style>
