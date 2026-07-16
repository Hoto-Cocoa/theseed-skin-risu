<template>
    <dropdown v-if="available">
        <template #toggle="{ show }">
            <button type="button" class="risu-user-toggle risu-notif-toggle" aria-label="알림" :aria-expanded="show">
                <icon name="bell" />
                <span v-if="notifications.length" class="risu-notif-badge">{{ notifications.length }}</span>
            </button>
        </template>
        <div class="dropdown-menu dropdown-menu-right risu-notif-menu">
            <template v-if="notifications.length">
                <template v-for="item in notifications" :key="item.uuid">
                    <nuxt-link v-if="linkOf(item)" :to="linkOf(item)" class="dropdown-item risu-notif-item">
                        <span class="risu-notif-title">{{ titleOf(item) }}</span>
                        <span v-if="previewOf(item)" class="risu-notif-preview">{{ previewOf(item) }}</span>
                        <local-date class="risu-notif-date" :date="item.createdAt" relative />
                    </nuxt-link>
                    <div v-else class="dropdown-item risu-notif-item">
                        <span class="risu-notif-title risu-notif-html" v-html="item.data" @click="onDynamicContentClick($event)"></span>
                        <local-date class="risu-notif-date" :date="item.createdAt" relative />
                    </div>
                </template>
            </template>
            <div v-else class="dropdown-item risu-notif-empty">새 알림이 없습니다.</div>
            <div class="dropdown-divider"></div>
            <nuxt-link to="/member/notifications" class="dropdown-item">알림 전체 보기</nuxt-link>
        </div>
    </dropdown>
</template>

<script>
import Common from '~/mixins/common';
import LocalDate from '~/components/localDate';
import Dropdown from './dropdown';
import Icon from './icon';

export default {
    mixins: [Common],
    components: {
        Dropdown,
        Icon,
        LocalDate
    },
    computed: {
        available() {
            /* 알림 배열은 신버전 엔진 전용 — 없으면 벨 자체를 감춘다 */
            return this.$store.state.session.account.type === 1
                && Array.isArray(this.$store.state.session.notifications);
        },
        notifications() {
            return this.$store.state.session.notifications ?? [];
        }
    },
    methods: {
        linkOf(item) {
            if (item.url) return item.url;
            if (item.thread?.url) return `/thread/${item.thread.url}`;
            return null;
        },
        titleOf(item) {
            /* NotificationTypes: 0 UserDiscuss, 1 Mention */
            if (item.type === 0) return `사용자 토론: ${item.thread?.topic ?? ''}`;
            if (item.type === 1) return `멘션: ${item.thread?.topic ?? ''}${item.comment?.id ? ` #${item.comment.id}` : ''}`;
            return '';
        },
        previewOf(item) {
            const text = item.comment?.contentHtml;
            return typeof text === 'string' ? this.removeHtmlTags(text) : '';
        }
    }
}
</script>
