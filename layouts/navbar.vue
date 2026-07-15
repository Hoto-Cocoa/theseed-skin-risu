<template>
    <header class="risu-navbar" :class="{ scrolled }">
        <nuxt-link class="risu-brand" to="/">
            <img v-if="logoImage" :src="logoImage" :alt="logoText">
            <template v-else>{{ logoText }}</template>
        </nuxt-link>
        <nav class="risu-nav">
            <nuxt-link class="risu-nav-link" to="/RecentChanges"><icon name="refresh" /><span class="hide-title">최근 변경</span></nuxt-link>
            <nuxt-link class="risu-nav-link" to="/RecentDiscuss"><icon name="comments" /><span class="hide-title">최근 토론</span></nuxt-link>
            <nuxt-link class="risu-nav-link" to="/random"><icon name="sparkle" /><span class="hide-title">임의 문서</span></nuxt-link>
            <dropdown>
                <template #toggle>
                    <a class="risu-nav-link" href="#" @click.prevent>
                        <icon name="sliders" /><span class="hide-title">도구</span> <span class="caret"></span>
                    </a>
                </template>
                <div class="dropdown-menu" role="menu">
                    <nuxt-link to="/Upload" class="dropdown-item">파일 올리기</nuxt-link>
                    <div class="dropdown-divider"></div>
                    <nuxt-link to="/NeededPages" class="dropdown-item">작성이 필요한 문서</nuxt-link>
                    <nuxt-link to="/OrphanedPages" class="dropdown-item">고립된 문서</nuxt-link>
                    <nuxt-link to="/OrphanedCategories" class="dropdown-item">고립된 분류</nuxt-link>
                    <nuxt-link to="/UncategorizedPages" class="dropdown-item">분류가 되지 않은 문서</nuxt-link>
                    <nuxt-link to="/OldPages" class="dropdown-item">편집된 지 오래된 문서</nuxt-link>
                    <nuxt-link to="/ShortestPages" class="dropdown-item">내용이 짧은 문서</nuxt-link>
                    <nuxt-link to="/LongestPages" class="dropdown-item">내용이 긴 문서</nuxt-link>
                    <nuxt-link to="/BlockHistory" class="dropdown-item">차단 내역</nuxt-link>
                    <nuxt-link to="/RandomPage" class="dropdown-item">RandomPage</nuxt-link>
                    <nuxt-link to="/License" class="dropdown-item">라이선스</nuxt-link>
                    <template v-if="$store.state.session.menus.length">
                        <div class="dropdown-divider"></div>
                        <nuxt-link v-for="m in $store.state.session.menus" :key="m.l" :to="m.l" class="dropdown-item">{{ m.t }}</nuxt-link>
                    </template>
                </div>
            </dropdown>
        </nav>
        <search-form />
        <div class="risu-user">
            <dropdown>
                <template #toggle>
                    <a class="risu-user-toggle" href="#" @click.prevent>
                        <img v-if="$store.state.session.gravatar_url" class="profile-img" :src="$store.state.session.gravatar_url">
                        <icon v-else name="user" />
                    </a>
                </template>
                <div class="dropdown-menu dropdown-menu-right">
                    <div v-if="$store.state.session.account.type === 1" class="username dropdown-item">
                        <b>{{ $store.state.session.account.name }}</b><br>Member
                    </div>
                    <div v-else-if="$store.state.session.account.type === 0" class="username dropdown-item">
                        <b>{{ $store.state.session.account.name }}</b><br>Please login!
                    </div>
                    <div class="dropdown-divider"></div>
                    <a href="#" class="dropdown-item" @click.prevent="openSettingModal">스킨 설정</a>
                    <a v-if="$store.state.currentTheme === 'light'" href="#" class="dropdown-item" @click.prevent="$store.commit('localConfigSetValue', { key: 'wiki.theme', value: 'dark' })">다크 테마로</a>
                    <a v-if="$store.state.currentTheme === 'dark'" href="#" class="dropdown-item" @click.prevent="$store.commit('localConfigSetValue', { key: 'wiki.theme', value: 'light' })">라이트 테마로</a>
                    <div class="dropdown-divider"></div>
                    <template v-if="$store.state.session.account.type === 1">
                        <nuxt-link to="/member/mypage" class="dropdown-item">내 정보</nuxt-link>
                        <nuxt-link :to="doc_action_link(user_doc($store.state.session.account.name), 'w')" class="dropdown-item">내 사용자 문서</nuxt-link>
                        <nuxt-link to="/member/starred_documents" class="dropdown-item">내 문서함</nuxt-link>
                        <div class="dropdown-divider"></div>
                    </template>
                    <template v-if="$store.state.session.account.uuid">
                        <nuxt-link class="dropdown-item" :to="contribution_link($store.state.session.account.uuid)">{{ discussContributionLink ? '내 문서 기여 목록' : '내 기여 목록' }}</nuxt-link>
                        <nuxt-link v-if="discussContributionLink" class="dropdown-item" :to="discussContributionLink">내 토론 기여 목록</nuxt-link>
                        <nuxt-link v-if="editRequestContributionLink" class="dropdown-item" :to="editRequestContributionLink">내 편집 요청 목록</nuxt-link>
                        <div class="dropdown-divider"></div>
                    </template>
                    <nuxt-link v-if="$store.state.session.account.type === 1" :to="{ path: '/member/logout', query: { redirect: $route.fullPath } }" class="dropdown-item">로그아웃</nuxt-link>
                    <nuxt-link v-else :to="{ path: '/member/login', query: { redirect: $route.fullPath } }" class="dropdown-item">로그인</nuxt-link>
                </div>
            </dropdown>
        </div>
    </header>
</template>

<script>
import Common from '~/mixins/common';
import Dropdown from '../components/dropdown';
import Icon from '../components/icon';
import SearchForm from './searchForm';
import SettingModal from '../components/settingModal';

export default {
    mixins: [Common],
    components: {
        Dropdown,
        Icon,
        SearchForm
    },
    data() {
        return {
            scrolled: false
        };
    },
    mounted() {
        window.addEventListener('scroll', this.onScroll, { passive: true });
        this.onScroll();
    },
    beforeUnmount() {
        window.removeEventListener('scroll', this.onScroll);
    },
    computed: {
        logoImage() {
            return this.$store.state.config['skin.risu.logo_image'] ?? this.$store.state.config['wiki.logo_url'];
        },
        discussContributionLink() {
            const uuid = this.$store.state.session.account.uuid;
            return typeof this.contribution_link_discuss === 'function' ? this.contribution_link_discuss(uuid) : null;
        },
        editRequestContributionLink() {
            const uuid = this.$store.state.session.account.uuid;
            return typeof this.contribution_link_edit_request === 'function' ? this.contribution_link_edit_request(uuid) : null;
        },
        logoText() {
            return this.$store.state.config['skin.risu.logo_text'] ?? this.$store.state.config['wiki.site_name'] ?? 'risu';
        }
    },
    methods: {
        onScroll() {
            this.scrolled = window.scrollY > 8;
        },
        openSettingModal() {
            this.$vfm.show({ component: SettingModal });
        }
    }
}
</script>
