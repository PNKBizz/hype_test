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
            li.autocomplete__input-wrapper
                input.autocomplete__input(
                    v-model="inputVal"
                    ref="autocompleteInput"
                    @click.stop=""
                    @keyup.enter="addTag(focusedOption)"
                    @keyup.up="arrowHandler('up')",
                    @keyup.down="arrowHandler('down')"
                    @focus.once="fetchOptions"
                    @focus="open"
                    @input="handleInput"
                )
                .autocomplete__options(v-show="showOptions")
                    ul.autocomplete__options-list
                        li.autocomplete__options-item(
                            v-if="optionsIsLoading"
                        ) Загружаем результаты...
                        li.autocomplete__options-item(
                            :class="{ focused: focusedOption === option }"
                            v-else
                            v-for="option in matchedOptions"
                            @mouseover="focusedOption = option"
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
        value: {
            type: Array,
            default: () => []
        },
        fetchUrl: String,
        matchedField: String,
        responseMapper: Function
    },
    data() {
        return {
            currentTags: this.value,
            currentOptions: this.options,
            fetchedOptions: null,
            showOptions: false,
            optionsIsLoading: false,
            inputVal: '',
            focusedOption: '',
            error: ''
        }
    },
    computed: {
        matchedOptions() {
            if (!this.inputVal) return [];
            const options = this.fetchedOptions || this.options;
            const field = typeof options[0] === 'object' && this.matchedField;
            const matched = options.filter(option => fuzzysearch(this.inputVal, field && option[field] || option));
            if (!matched.length) this.close();
            return matched;
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
            if (!option) return;
            this.currentTags = this.currentTags.concat([option]);
            this.inputVal = '';
            this.close();
            this.$refs.autocompleteInput.focus();
            this.$emit('input', this.currentTags);
        },
        deleteTag(deletedTag) {
            const field = this.matchedField;
            const deleted = field && deletedTag[field] || deletedTag;
            this.currentTags = this.currentTags.filter(tag => (field && tag[field] || tag) !== deleted );
        },
        close() {
            this.showOptions = false;
            this.focusedOption = '';
        },
        open() {
            this.showOptions = !!this.inputVal;
        },
        arrowHandler(key) {
            if (this.showOptions && this.matchedOptions.length) {
                const currentIndex = this.matchedOptions.findIndex(option => option === this.focusedOption);
                if (key === 'up' && currentIndex > 0) {
                    this.focusedOption = this.matchedOptions[currentIndex - 1]; 
                } else if (key === 'down' && currentIndex < (this.matchedOptions.length - 1)) {
                    this.focusedOption = this.matchedOptions[currentIndex + 1]; 
                }
            }
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
        display: flex;
        position: relative;
        border-bottom: 1px solid #777;
        flex-wrap: wrap;

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
            flex-wrap: wrap;
            flex-grow: 1;

            &-item {
                padding: 5px 25px 5px 10px;
                background-color: rgb(35, 123, 255);
                color: white;
                margin: 5px;
                position: relative;
                border-radius: 3px;

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
            width: 10px;

            &-wrapper {
                position: relative;
                flex-grow: 1;
                display: flex;
            }
        }

        &__options {
            position: absolute;
            z-index: 99;
            top: calc(100% + 10px);
            left: -50%;
            width: 100%;
            max-height: 300px;
            display: flex;
            flex-direction: column;
            align-items: center;

            &:before {
                content: '';
                width: 0;
                height: 0;
                border: 5px solid transparent;
                border-bottom-color: white;
                position: absolute;
                top: -10px;
            }

            &-list {
                overflow-y: auto;
                padding-left: 10px;
                background-color: white;
                border-radius: 5px;
                position: relative;
            }

            &-item {
                padding: 10px 20px;
                margin:
                 5px 0;

                &:first-child, &:last-child {
                    margin: 0;
                }

                &.focused {
                    background-color: rgb(35, 123, 255);
                    color: white;
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

