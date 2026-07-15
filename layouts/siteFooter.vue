<template>
    <footer id="bottom" class="risu-footer">
        <ul v-if="$store.state.page.viewName === 'wiki' && $store.state.page.data.date" class="footer-info">
            <li v-if="$store.state.page.data.rev" class="footer-info-lastmod">이 리비전은 <local-date :date="$store.state.page.data.date" />에 편집되었습니다.</li>
            <li v-else class="footer-info-lastmod">이 문서는 <local-date :date="$store.state.page.data.date" />에 마지막으로 편집되었습니다.</li>
            <li class="footer-info-copyright" v-html="$store.state.page.data.copyright_text ?? $store.state.config['wiki.copyright_text']" @click="onDynamicContentClick($event)" />
        </ul>
        <ul class="footer-places" v-html="$store.state.config['skin.risu.footer_html'] || $store.state.config['wiki.footer_text']" @click="onDynamicContentClick($event)" />
        <ul class="footer-icons">
            <li class="footer-poweredbyico">
                <nuxt-link to="/License">risu</nuxt-link> | <a href="//theseed.io/">the seed</a>
            </li>
        </ul>
    </footer>
</template>

<script>
import Common from '~/mixins/common';
import LocalDate from '~/components/localDate';

export default {
    mixins: [Common],
    components: {
        LocalDate
    }
}
</script>
