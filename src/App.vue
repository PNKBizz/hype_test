<template lang="pug">
    #app
        autocomplete.autocomplete(
            :fetchUrl="fetchUrl",
            :responseMapper="logger"
            matchedField="value"
        )
            span(slot="tagLayout", slot-scope="{ tag }") {{ `@${tag.value}` }}
            span(slot="optionLayout", slot-scope="{ option }") {{ `@${option.value}` }}
</template>

<script>
import autocomplete from './components/autocomplete.vue';

export default {
    name: 'app',
    data() {
        return {
            fetchUrl: 'http://www.mocky.io/v2/5ac329f83000004900f46d02'
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
    }

    #app {
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .autocomplete {
        width: 600px;
    }
</style>
