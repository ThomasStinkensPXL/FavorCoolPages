<script setup>
import {Configuration, OpenAIApi} from "openai";
// import 'dotenv/config'
</script>

<template>
  <main>
    Hallo en welkom bij deze epische bot... die maar gebasseerd is op 1 bestand.
    <br>
    <input v-model="question">
    <button v-on:click="askQuestion(question)">Submit</button>
  </main>
</template>

<script>
import {Configuration, OpenAIApi} from "openai";

export default{
  name: "App",
  components: {

  },
  data() {
    return {
      question: "",
      openAIApi: new OpenAIApi()
    }
  },
  methods: {
    submitQueryToLLM: async function(question) {
      if (question.trim() !== "") {
        return await this.openAIApi.createChatCompletion({
          model: "gpt-3.5-turbo-16k",
          messages: [{role: "user", content: "What day is it today?"}],
        });
      }
    },
    submitQueryToVectorDatabase: async function (question) {
      let vectors = await this.embedQuery(question);
      let response = await fetch(import.meta.env.VITE_PINECONE_API_QUERRY_ENDPOINT, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Api-Key": import.meta.env.VITE_PINECONE_API_KEY,
        },
        body: JSON.stringify({
          "vector": vectors,
          "topK": 3,
          "includeMetadata": true
        })
      }).then(response => {
        if (response.status === 200) {
          return response.json();
        }
      }).catch(error => console.log(error.message))
      console.log(response)
    },
    askQuestion: function(question) {
      this.setupOpenAI();
      this.submitQueryToVectorDatabase(question);
      // this.submitQueryToLLM(question);
    },
    setupOpenAI: function() {
      let config = new Configuration({
        apiKey: import.meta.env.VITE_OPENAI_API_KEY
      });

      this.openAIApi = new OpenAIApi(config);
    },
    embedQuery: async function (question) {
      let response = await this.openAIApi.createEmbedding({model: "text-embedding-ada-002", input: `${question}`});
      // Dit werkt WEL (dacht van niet), er was een probleem met een ambiguous import. De API request hieronder heb ik eig niet getest omdat ik dit eerder had gevonden. Maar de API request hoort te werken!

      return response.data.data[0].embedding

      // return await fetch('https://api.openai.com/v1/embeddings', {
      //   headers: {
      //     'Content-Type': 'application/json',
      //     Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
      //   },
      //   method: 'POST',
      //   body: JSON.stringify({ input: question, model: "text-embedding-ada-002"}),
      // });

    }
  }
}

</script>

<style scoped>
header {
  line-height: 1.5;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>