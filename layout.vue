<template>
    <div class="risu" :class="rootClass" :style="skinConfig">
        <div id="top"></div>
        <navbar />
        <div class="risu-wrap" :class="{ 'hide-sidebar': sidebarMode !== 'right' }">
            <main class="risu-main">
                <div v-if="$store.state.config['wiki.sitenotice']" id="site-notice" class="risu-notice">
                    <span v-html="$store.state.config['wiki.sitenotice']" @click="onDynamicContentClick($event)" />
                </div>
                <div class="risu-content">
                    <div class="risu-content-header">
                        <content-tool @onClickEditBtn="showEditMessage" />
                        <div class="title">
                            <h1 v-if="$store.state.page.data.document && $store.state.page.viewName !== 'error'">
                                <nuxt-link :to="doc_action_link($store.state.page.data.document, 'w')"><span v-if="$store.state.page.data.document.forceShowNamespace !== false" class="namespace">{{ $store.state.page.data.document.namespace }}:</span>{{ $store.state.page.data.document.title }}</nuxt-link>
                                <small v-if="$store.state.page.viewName === 'edit_edit_request' || $store.state.page.viewName === 'edit_request'">(편집 요청)</small>
                                <small v-else-if="$store.state.page.viewName === 'edit' && $store.state.page.data.body.section">(r{{ $store.state.page.data.body.baserev }} 문단 편집)</small>
                                <small v-else-if="$store.state.page.viewName === 'edit' && $store.state.page.data.body.baserev === '0'">(새 문서 생성)</small>
                                <small v-else-if="$store.state.page.viewName === 'edit'">(r{{ $store.state.page.data.body.baserev }} 편집)</small>
                                <small v-else-if="$store.state.page.viewName === 'history'">(역사)</small>
                                <small v-else-if="$store.state.page.viewName === 'backlink'">(역링크)</small>
                                <small v-else-if="$store.state.page.viewName === 'move'">(이동)</small>
                                <small v-else-if="$store.state.page.viewName === 'delete'">(삭제)</small>
                                <small v-else-if="$store.state.page.viewName === 'acl'">(ACL)</small>
                                <small v-else-if="$store.state.page.viewName === 'thread'">(토론)</small>
                                <small v-else-if="$store.state.page.viewName === 'thread_list'">(토론 목록)</small>
                                <small v-else-if="$store.state.page.viewName === 'thread_list_close'">(닫힌 토론)</small>
                                <small v-else-if="$store.state.page.viewName === 'edit_request_close'">(닫힌 편집 요청)</small>
                                <small v-else-if="$store.state.page.viewName === 'diff'">(비교)</small>
                                <small v-else-if="$store.state.page.viewName === 'revert' && $store.state.page.data.rev">(r{{ $store.state.page.data.rev }}로 되돌리기)</small>
                                <small v-else-if="$store.state.page.viewName === 'raw' && $store.state.page.data.rev">(r{{ $store.state.page.data.rev }} RAW)</small>
                                <small v-else-if="$store.state.page.viewName === 'blame' && $store.state.page.data.rev">(r{{ $store.state.page.data.rev }} Blame)</small>
                                <small v-else-if="$store.state.page.viewName === 'wiki' && $store.state.page.data.rev">(r{{ $store.state.page.data.rev }} 판)</small>
                            </h1>
                            <h1 v-else>{{ $store.state.page.title }}</h1>
                        </div>
                    </div>
                    <div class="wiki-article">
                        <alert v-if="isShowACLMessage && $store.state.page.data.edit_acl_message" @close="isShowACLMessage = false" error closable>
                            <span v-html="$store.state.page.data.edit_acl_message" @click="onDynamicContentClick($event)"></span>
                            <span v-if="requestable"><br v-if="$store.state.page.data.edit_acl_message.includes('\n')"> 대신 <nuxt-link :to="doc_action_link($store.state.page.data.document, 'new_edit_request')">편집 요청</nuxt-link>을 생성할 수 있습니다.</span>
                        </alert>
                        <alert v-if="$store.state.session.user_document_discuss && $store.state.localConfig['wiki.hide_user_document_discuss'] !== $store.state.session.user_document_discuss" @close="$store.commit('localConfigSetValue', { key: 'wiki.hide_user_document_discuss', value: $store.state.session.user_document_discuss })" closable theme="primary">
                            현재 진행 중인 <nuxt-link :to="doc_action_link(user_doc($store.state.session.account.name), 'discuss')">사용자 토론</nuxt-link>이 있습니다.
                        </alert>
                        <alert v-if="$store.state.page.viewName === 'notfound' && $store.state.page.data.document.namespace === '문서'" style="line-height: 2.1rem;">
                            '{{ $store.state.page.title }}'을(를) 검색하시겠습니까?
                            <div class="float-right"><seed-link-button :to="'/Search?q=' + $store.state.page.title">검색</seed-link-button></div>
                            <div class="clearfix"></div>
                        </alert>
                        <nuxt />
                        <div v-if="$store.state.page.viewName === 'license'">
                            <h2>risu skin license</h2>
                            <pre>{{ License }}</pre>
                        </div>
                        <div class="clearfix"></div>
                    </div>
                </div>
                <site-footer />
                <div v-if="sidebarMode === 'footer'" class="footer-recent">
                    <div class="live-recent">
                        <div class="live-recent-header">최근 변경</div>
                        <recent-card :limit="8" />
                        <div class="live-recent-footer">
                            <nuxt-link to="/RecentChanges" title="최근 변경내역"><span class="label">더 보기</span></nuxt-link>
                        </div>
                    </div>
                </div>
            </main>
            <aside class="risu-sidebar">
                <div class="live-recent">
                    <div class="live-recent-header">최근 변경</div>
                    <recent-card />
                    <div class="live-recent-footer">
                        <nuxt-link to="/RecentChanges" title="최근 변경내역"><span class="label">더 보기</span></nuxt-link>
                    </div>
                </div>
            </aside>
        </div>
        <div class="scroll-buttons">
            <nuxt-link class="scroll-toc" to="#toc" title="목차"><icon name="list" /></nuxt-link>
            <nuxt-link class="scroll-top" to="#top" title="맨 위로"><icon name="arrow-up" /></nuxt-link>
            <nuxt-link class="scroll-bottom" to="#bottom" title="맨 아래로"><icon name="arrow-down" /></nuxt-link>
        </div>
    </div>
</template>

<style>
@import "./css/tokens.css";
@import "./css/default.css";
@import "./css/mobile.css";
@import "./css/dark.css";
</style>

<script>
import Common from '~/mixins/common';
import Alert from '~/components/alert';
import SeedLinkButton from '~/components/seedLinkButton';
import Navbar from './layouts/navbar';
import RecentCard from './layouts/recentCard';
import ContentTool from './layouts/contentTool';
import SiteFooter from './layouts/siteFooter';
import Icon from './components/icon';
import License from "raw-loader!./LICENSE";

export default {
    mixins: [Common],
    components: {
        Alert,
        SeedLinkButton,
        Navbar,
        RecentCard,
        ContentTool,
        SiteFooter,
        Icon
    },
    data() {
        return {
            License,
            isShowACLMessage: false
        };
    },
    watch: {
        $route() {
            this.isShowACLMessage = false;
        }
    },
    head() {
        return {
            meta: [{ name: 'theme-color', content: this.brand_color }]
        };
    },
    computed: {
        brand_color() {
            return this.selectByTheme(this.$store.state.config['skin.risu.brand_color'] ?? '#FFC0CB', '#211419');
        },
        sidebarMode() {
            const value = this.$store.state.localConfig['risu.sidebar'];
            return value === 'hide' || value === 'footer' ? value : 'right';
        },
        rootClass() {
            return {
                'navbar-fixed': this.$store.state.localConfig['risu.fixed_navbar'] !== false,
                'wide': this.$store.state.localConfig['risu.wide_mode'] === true
            };
        },
        skinConfig() {
            const brand = this.$store.state.config['skin.risu.brand_color'];
            const config = {
                '--risu-logo-width': this.$store.state.config['skin.risu.logo_width'],
                '--brand-color-1': 'var(--risu-rose)',
                '--brand-color-2': 'var(--risu-rose)',
                '--brand-bright-color-1': 'var(--risu-pink-soft)',
                '--brand-bright-color-2': 'var(--risu-pink-soft)',
                '--text-color': this.selectByTheme('#43333A', '#F0DEE4'),
                '--article-background-color': this.selectByTheme('#fff', '#2B1C22'),
            };
            if (brand && this.$store.state.currentTheme !== 'dark') {
                config['--risu-pink'] = brand;
                config['--risu-pink-soft'] = this.lightenColor(brand, 35);
                config['--risu-pink-mist'] = this.lightenColor(brand, 65);
                config['--risu-page-bg'] = this.lightenColor(brand, 80);
                config['--risu-border'] = this.lightenColor(brand, 20);
                config['--risu-border-soft'] = this.lightenColor(brand, 50);
                config['--risu-rose'] = this.darkenColor(brand, 45);
                config['--risu-rose-hover'] = this.darkenColor(brand, 55);
                config['--risu-rose-deep'] = this.darkenColor(brand, 70);
            }
            return config;
        },
        requestable() {
            return this.$store.state.page.data.editable === true && this.$store.state.page.data.edit_acl_message && this.$store.state.page.viewName !== 'notfound';
        }
    },
    methods: {
        showEditMessage() {
            if (this.isShowACLMessage) {
                this.$router.push(this.doc_action_link(this.$store.state.page.data.document, this.requestable ? 'new_edit_request' : 'edit'));
            }
            else {
                this.isShowACLMessage = true;
            }
        },
        darkenColor(hex, percent = 50) {
            let r = parseInt(hex.substring(1, 3), 16);
            let g = parseInt(hex.substring(3, 5), 16);
            let b = parseInt(hex.substring(5, 7), 16);

            r = Math.round(r * (1 - percent / 100));
            g = Math.round(g * (1 - percent / 100));
            b = Math.round(b * (1 - percent / 100));

            return "#" + ((r < 16 ? "0" : "") + r.toString(16)) + ((g < 16 ? "0" : "") + g.toString(16)) + ((b < 16 ? "0" : "") + b.toString(16));
        },
        lightenColor(hex, percent = 50) {
            let r = parseInt(hex.substring(1, 3), 16);
            let g = parseInt(hex.substring(3, 5), 16);
            let b = parseInt(hex.substring(5, 7), 16);

            r = Math.round(r + (255 - r) * (percent / 100));
            g = Math.round(g + (255 - g) * (percent / 100));
            b = Math.round(b + (255 - b) * (percent / 100));

            return "#" + ((r < 16 ? "0" : "") + r.toString(16)) + ((g < 16 ? "0" : "") + g.toString(16)) + ((b < 16 ? "0" : "") + b.toString(16));
        },
        selectByTheme(light, dark) {
            return this.$store.state.currentTheme === 'dark' ? dark : light;
        }
    }
}
</script>
