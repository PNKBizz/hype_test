<template lang="pug">
    #app
        autocomplete.autocomplete(
            v-model="tags"
            :fetchUrl="fetchUrl",
            :responseMapper="logger"
            matchedField="value"
        )
            span(slot="tagLayout", slot-scope="{ tag }") {{ `@${tag.value}` }}
            span(slot="optionLayout", slot-scope="{ option }") {{ `@${option.value}` }}
        
        .output current tags: {{ tags }}
</template>

<script>
import autocomplete from './components/autocomplete.vue';

export default {
    name: 'app',
    data() {
        return {
            fetchUrl: 'http://www.mocky.io/v2/5ac329f83000004900f46d02',
            tags: []
        };
    },
    components: { autocomplete },
    methods: {
        logger(data) {
            return data.map(item => {
                console.log(item);
                return item;
            });
        }
    }
};
</script>

<style lang="scss">
    body {
        width: 100%;
        height: 100vh;
        margin: 0;
        background-color: #DDD;
    }

    #app {
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .autocomplete {
        width: 500px;
    }

    .output {
        margin-top: 20px;
        padding: 20px;
        width: 500px;
        background-color: white;
        border-radius: 10px;
    }
</style>
