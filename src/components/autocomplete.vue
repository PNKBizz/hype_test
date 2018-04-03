<template lang="pug">
    section.autocomplete
        ul.autocomplete__tags
            li.autocomplete__tags-item(
                v-for="tag in currentTags"
            )
                span(v-if="isTagSlotEmpty") {{ tag }}
                slot(name="tagLayout", :tag="tag")
                span.close-icon(
                    @click.stop="deleteTag(tag)"
                ) &#10006;
        input.autocomplete__input(
            v-model="inputVal"
            @click.stop=""
            @keyup.enter="addTagByEnter"
            @focus.once="fetchOptions"
            @focus="open"
            @input="handleInput"
        )
        ul.autocomplete__options(v-show="showOptions")
            li.autocomplete__options-item(
                v-if="optionsIsLoading"
            ) Загружаем результаты...
            li.autocomplete__options-item(
                v-else
                v-for="option in matchedOptions"
                @click.stop="addTag(option)"
            )
                span(v-if="isOptionSlotEmpty") {{ option }}
                slot(name="optionLayout", :option="option")
        span.error(v-if="error") {{ error }} 
</template>

<script>
import fuzzysearch from 'fuzzysearch';
import axios from 'axios';

export default {
    props: {
        options: {
            type: Array,
            default: () => []
        },
        tags: {
            type: Array,
            default: () => []
        },
        fetchUrl: String,
        matchedField: String,
        responseMapper: Function
    },
    data() {
        return {
            currentTags: this.tags,
            currentOptions: this.options,
            fetchedOptions: null,
            showOptions: false,
            optionsIsLoading: false,
            inputVal: '',
            error: ''
        }
    },
    computed: {
        matchedOptions() {
            if (!this.inputVal) return [];
            const options = this.fetchedOptions || this.options;
            const field = typeof options[0] === 'object' && this.matchedField;
            return options.filter(option => fuzzysearch(this.inputVal, field && option[field] || option));
        },
        isOptionSlotEmpty() { return !this.$scopedSlots.optionLayout; },
        isTagSlotEmpty() { return !this.$scopedSlots.tagLayout; }
    },
    methods: {
        handleInput({ target: { value } }) {
            if (!value) {
                this.close();
            } else {
                this.showOptions = true;
            }
        },
        fetchOptions() {
            this.optionsIsLoading = true;
            axios.get(this.fetchUrl).then(({ data }) => {
                this.fetchedOptions = this.responseMapper ? this.responseMapper(data) : data;
                this.error = '';
                this.optionsIsLoading = false;
            }).catch(err => {
                this.optionsIsLoading = false;
                this.fetchedOptions = [];
                this.error = 'Ошибка загрузки данных.'
            });
        },
        addTag(option) {
            this.currentTags = this.currentTags.concat([option]);
        },
        deleteTag(deletedTag) {
            const field = this.matchedField;
            const deleted = field && deletedTag[field] || deletedTag;
            this.currentTags = this.currentTags.filter(tag => (field && tag[field] || tag) !== deleted );
        },
        close() {
            this.showOptions = false;
        },
        open() {
            this.showOptions = !!this.inputVal;
        }
    },
    mounted() {
        if (document) {
            document.addEventListener('click', this.close);
        }
    },
    beforeDestroy() {
        if (document) {
            document.removeEventListener('click', this.close);
        }
    }
}
</script>

<style lang="scss" scoped>
    .autocomplete {
        max-width: 500px;
        display: flex;
        position: relative;
        padding-bottom: 5px;
        border-bottom: 1px solid #777;

        ul {
            list-style: none;
            margin: 0;
            padding: 0;

            li {
                cursor: pointer;
            }
        }

        &__tags {
            display: flex;

            &-item {
                padding: 5px 25px 5px 10px;
                background-color: #DDD;
                color: #777;
                margin: 0 5px;
                position: relative;
                border-radius: 5px;

                .close-icon {
                    position: absolute;
                    right: 7px;
                    font-size: .7em;
                }
            }
        }

        &__input {
            flex-grow: 1;
            border: none;
            background-color: transparent;
            outline: none;
            font-size: 1.2em;
        }

        &__options {
            position: absolute;
            z-index: 99;
            top: 100%;
            width: 100%;
            padding-left: 10px;
            background-color: rgba($color: #fff, $alpha: .9);
            max-height: 300px;
            overflow-y: auto;

            &-item {
                padding: 10px 20px;
                margin: 5px 0;

                &:first-child, &:last-child {
                    margin: 0;
                }

                &:hover {
                    background-color: #DDD;
                }
            }
        }

        .error {
            position: absolute;
            top: calc(100% + 10px);
            color: red;
        }
    }
</style>

