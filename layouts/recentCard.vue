<template>
    <div class="live-recent-content">
        <ul class="live-recent-list">
            <template v-if="recent.length === 0 && !loaded">
                <li v-for="i in 10" :key="i"><span class="recent-item">&nbsp;</span></li>
            </template>
            <li v-else-if="recent.length === 0" class="live-recent-empty">
                아직 표시할 문서가 없습니다.
            </li>
            <template v-else>
                <template v-for="(r, i) in recent" :key="i">
                    <li v-if="i < limit">
                        <nuxt-link class="recent-item" :class="{ removed: r.status === 'delete' }" :to="doc_action_link(r.document, 'w')">
                            [<local-date :date="r.date" :format="getDateType(r.date)" />] {{ r.document }}
                        </nuxt-link>
                    </li>
                </template>
            </template>
        </ul>
    </div>
</template>

<script>
import RecentCardMixin from '~/mixins/recentCard';
import LocalDate from '~/components/localDate';

export default {
    mixins: [RecentCardMixin],
    components: {
        LocalDate
    },
    props: {
        limit: {
            type: Number,
            default: 15
        }
    },
    data() {
        return {
            loaded: false
        };
    },
    async mounted() {
        await this.updateSidebar();
        this.loaded = true;
    },
    methods: {
        getDateType(date) {
            const now = Math.floor((new Date()).getTime() / 1000);
            return (now - 86400) > date ? 'Y/m/d' : 'H:i:s';
        }
    }
}
</script>
