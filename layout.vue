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
            <aside v-if="sidebarMode === 'right'" class="risu-sidebar">
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
@import "./css/thetree.css";
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
        const styles = [];
        if (this.brandOverrideCss) styles.push({ innerHTML: this.brandOverrideCss });
        if (this.fontCss) styles.push({ innerHTML: this.fontCss });
        return {
            meta: [{ name: 'theme-color', content: this.brand_color }],
            link: this.fontLinks,
            style: styles
        };
    },
    computed: {
        brand_color() {
            const brand = this.resolvedBrand;
            if (brand && /^#[0-9a-fA-F]{6}$/.test(brand)) {
                const { h, s } = this.hexToHsl(brand);
                return this.selectByTheme(brand, this.hslToHex(h, (s / 100) * 25, 10));
            }
            return this.selectByTheme('#FFC0CB', '#211419');
        },
        sidebarMode() {
            const value = this.$store.state.localConfig['risu.sidebar'];
            return value === 'hide' || value === 'footer' ? value : 'right';
        },
        rootClass() {
            return {
                'navbar-floating': this.$store.state.localConfig['risu.floating_navbar'] !== false,
                'wide': this.$store.state.localConfig['risu.wide_mode'] === true
            };
        },
        skinConfig() {
            /* 테마 의존 값을 여기(인라인 스타일)에 넣으면 SSR 첫 페인트와
             * 하이드레이션 사이에 색이 튄다. 테마별 값은 전부 tokens.css가 담당하고,
             * 인라인은 테마와 무관한 값만 유지한다. */
            return {
                '--risu-logo-width': this.$store.state.config['skin.risu.logo_width'],
            };
        },
        resolvedBrand() {
            /* 우선순위: 사용자 설정(프리셋/커스텀) > 위키 설정 > 기본 핑크(오버라이드 없음) */
            const presets = {
                pink: '#FFC0CB',
                lavender: '#D9C6F2',
                mint: '#BFE8D2',
                sky: '#BFD9F2',
                peach: '#FFD2B8'
            };
            const choice = this.$store.state.localConfig['risu.color'];
            if (presets[choice]) return presets[choice];
            if (choice === 'custom') {
                const custom = this.$store.state.localConfig['risu.color_custom'];
                if (/^#[0-9a-fA-F]{6}$/.test(custom)) return custom;
            }
            return this.$store.state.config['skin.risu.brand_color'];
        },
        brandOverrideCss() {
            const brand = this.resolvedBrand;
            if (!brand || !/^#[0-9a-fA-F]{6}$/.test(brand)) return '';
            /* 기본 핑크 팔레트의 채도·명도 레시피에 브랜드 색상(hue)을 갈아끼운다.
             * 단순 darken/lighten은 채도가 죽어 로즈가 탁해진다.
             * 브랜드 채도(sb)는 틴트에 그대로, 진한 색에는 sqrt로 완만하게 반영.
             * body 레벨에 선언해 teleport된 모달/팝업까지 상속시킨다. */
            const { h, s, l } = this.hexToHsl(brand);
            const sb = s / 100;
            const strong = Math.sqrt(sb);
            const tint = (sat, light) => this.hslToHex(h, sat * sb, light);
            const deep = (sat, light) => this.hslToHex(h, sat * strong, light);

            const soft = tint(100, 92.5);
            const rose = deep(51, 51);
            const roseDeep = deep(51, 29);
            const text = deep(14, 23);
            const muted = deep(11, 49);
            const shadow = deep(51, 57);
            /* 내비바·btn-primary·라벨 등은 --risu-pink(브랜드 원색)를 배경으로 그대로 쓴다.
             * rose-deep은 "밝은 배경 위 텍스트" 레시피라 브랜드 자체가 어두우면
             * (예: 짙은 다크레드) 배경도 텍스트도 둘 다 어두워져 안 보인다.
             * 브랜드 자체 명도(l)가 낮을 때만 밝은 톤으로 전환한다. */
            const onBrand = l < 55 ? deep(37, 91) : roseDeep;
            const r = parseInt(shadow.substring(1, 3), 16);
            const g = parseInt(shadow.substring(3, 5), 16);
            const b = parseInt(shadow.substring(5, 7), 16);
            const lightVars = [
                `--risu-pink:${brand}`,
                `--risu-pink-soft:${soft}`,
                `--risu-pink-mist:${tint(100, 96)}`,
                `--risu-page-bg:${tint(100, 98)}`,
                `--risu-border:${tint(69, 88)}`,
                `--risu-border-soft:${tint(75, 94)}`,
                `--risu-rose:${rose}`,
                `--risu-rose-hover:${deep(56, 43)}`,
                `--risu-rose-deep:${roseDeep}`,
                `--risu-on-brand:${onBrand}`,
                `--risu-text:${text}`,
                `--risu-text-muted:${muted}`,
                `--risu-shadow-sm:0 2px 8px rgba(${r},${g},${b},0.10)`,
                `--risu-shadow-md:0 6px 20px rgba(${r},${g},${b},0.14)`,
                `--risu-shadow-lg:0 10px 32px rgba(${r},${g},${b},0.18)`,
                /* :root에 선언된 제네릭 매핑은 그 시점 값으로 굳으므로 여기서 재선언 */
                `--text-color:${text}`,
                `--text-secondary-color:${muted}`,
                `--brand-color-1:${rose}`,
                `--brand-color-2:${rose}`,
                `--brand-bright-color-1:${soft}`,
                `--brand-bright-color-2:${soft}`
            ];

            /* 다크 레시피 (tokens.css 다크 블록의 HSL 분해값 기준) */
            const dSoft = tint(22, 24);
            const dRose = deep(100, 81);
            const dText = deep(37, 91);
            const dMuted = deep(18, 62);
            const dCard = tint(10, 14);
            const dRoseDeep = deep(100, 91);

            /* 엔진 컴포넌트가 테마를 스스로 고르는 light/dark 변수 쌍
             * (thetree.css :root 고정 선언을 덮는다) — 양쪽 블록에 모두 공급 */
            const themePair = [
                `--light-text-color:${text}`,
                `--dark-text-color:${dText}`,
                `--light-article-background-color:#FFFFFF`,
                `--dark-article-background-color:${dCard}`,
                `--light-text-secondary-color:${muted}`,
                `--dark-text-secondary-color:${dMuted}`
            ];
            const darkVars = [
                `--risu-pink:${tint(23, 18)}`,
                `--risu-pink-soft:${dSoft}`,
                `--risu-pink-mist:${tint(10, 16)}`,
                `--risu-page-bg:${tint(12, 10)}`,
                `--risu-card-bg:${dCard}`,
                `--risu-border:${tint(12, 25)}`,
                `--risu-border-soft:${tint(10, 19)}`,
                `--risu-rose:${dRose}`,
                `--risu-rose-hover:${deep(100, 86)}`,
                `--risu-rose-deep:${dRoseDeep}`,
                `--risu-on-brand:${dRoseDeep}`,
                `--risu-text:${dText}`,
                `--risu-text-muted:${dMuted}`,
                `--text-color:${dText}`,
                `--text-secondary-color:${dMuted}`,
                `--article-background-color:${dCard}`,
                `--brand-color-1:${dRose}`,
                `--brand-color-2:${dRose}`,
                `--brand-bright-color-1:${dSoft}`,
                `--brand-bright-color-2:${dSoft}`
            ];

            /* 다크는 tokens.css의 body.theseed-dark-mode(0,1,1)를 이기도록 클래스 중복으로 특이도 확보 */
            return `body:not(.theseed-dark-mode){${[...lightVars, ...themePair].join(';')}}`
                + `body.theseed-dark-mode.theseed-dark-mode{${[...darkVars, ...themePair].join(';')}}`;
        },
        selectedFont() {
            const value = this.$store.state.localConfig['risu.font'];
            return ['pretendard', 'noto', 'custom'].includes(value) ? value : '';
        },
        fontLinks() {
            if (this.selectedFont === 'pretendard') return [
                { rel: 'stylesheet', href: 'https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable-dynamic-subset.min.css' }
            ];
            if (this.selectedFont === 'noto') return [
                { rel: 'stylesheet', href: 'https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;600;700&display=swap' }
            ];
            return [];
        },
        fontCss() {
            /* :root:root — tokens.css의 :root 선언보다 특이도를 높여 로드 순서와 무관하게 적용 */
            if (this.selectedFont === 'noto') return ':root:root{--risu-font:"Noto Sans KR","Malgun Gothic","맑은 고딕","Apple SD Gothic Neo",sans-serif}';
            if (this.selectedFont === 'custom') {
                /* <style>에 들어가므로 폰트명에 쓰이는 문자만 허용 */
                const custom = (this.$store.state.localConfig['risu.font_custom'] ?? '').replace(/[^\w\s가-힣.,-]/g, '').trim();
                if (custom) return `:root:root{--risu-font:"${custom}","Malgun Gothic","맑은 고딕","Apple SD Gothic Neo",sans-serif}`;
            }
            /* pretendard는 tokens.css 기본 스택 최상단에 이미 있으므로 웹폰트 로드만으로 충분 */
            return '';
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
        hexToHsl(hex) {
            const r = parseInt(hex.substring(1, 3), 16) / 255;
            const g = parseInt(hex.substring(3, 5), 16) / 255;
            const b = parseInt(hex.substring(5, 7), 16) / 255;
            const max = Math.max(r, g, b);
            const min = Math.min(r, g, b);
            const l = (max + min) / 2;
            let h = 0, s = 0;
            if (max !== min) {
                const d = max - min;
                s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
                switch (max) {
                    case r: h = (g - b) / d + (g < b ? 6 : 0); break;
                    case g: h = (b - r) / d + 2; break;
                    default: h = (r - g) / d + 4;
                }
                h *= 60;
            }
            return { h, s: s * 100, l: l * 100 };
        },
        hslToHex(h, s, l) {
            s /= 100;
            l /= 100;
            const k = n => (n + h / 30) % 12;
            const a = s * Math.min(l, 1 - l);
            const f = n => l - a * Math.max(-1, Math.min(k(n) - 3, Math.min(9 - k(n), 1)));
            const to = x => Math.round(x * 255).toString(16).padStart(2, '0');
            return '#' + to(f(0)) + to(f(8)) + to(f(4));
        },
        selectByTheme(light, dark) {
            return this.$store.state.currentTheme === 'dark' ? dark : light;
        }
    }
}
</script>
