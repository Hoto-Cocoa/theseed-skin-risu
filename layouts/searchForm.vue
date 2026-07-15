<template>
    <form class="risu-search" @submit.prevent>
        <div class="input-search">
            <input type="search" name="q" placeholder="검색" accesskey="f" autocomplete="off" v-on:input="searchText = $event.target.value" v-model="searchTextModel" @blur="blur" @focus="focus" @input="inputChange" @keydown.enter="keyEnter" @keydown.tab="keyEnter" @keydown.up="keyUp" @keydown.down="keyDown">
            <div class="risu-search-buttons">
                <button type="submit" name="fulltext" value="검색" title="검색" @click="onClickSearch"><icon name="search" /></button>
                <button type="submit" name="fulltext" value="보기" title="이동" @click="onClickMove"><icon name="arrow-right" /></button>
            </div>
            <div v-if="show" class="v-autocomplete-list">
                <div class="v-autocomplete-list-item" v-for="(item, i) in internalItems" @click="onClickItem(item)" v-bind:key="i" :class="{ 'v-autocomplete-item-active': i === cursor }" @mouseover="cursor = i">
                    <div>{{ item }}</div>
                </div>
            </div>
        </div>
    </form>
</template>

<script>
import AutocompleteMixin from '~/mixins/autocomplete';
import Common from '~/mixins/common';
import Icon from '../components/icon';

export default {
    mixins: [AutocompleteMixin],
    components: {
        Icon
    },
    methods: {
        onClickSearch() {
            if (!this.searchText) return;
            this.$router.push('/Search?q=' + encodeURIComponent(this.searchText));
        },
        onClickMove() {
            if (!this.searchText) return;
            this.$router.push(Common.methods.doc_action_link(this.searchText, 'w'));
        }
    },
    watch: {
        $route() {
            if (this.$store.state.localConfig['risu.reset_search_on_move'] !== false) this.reset();
        }
    }
}
</script>
