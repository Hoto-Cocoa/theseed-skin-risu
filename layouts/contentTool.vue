<template>
    <div v-if="main.length" class="content-tools">
        <div class="btn-group">
            <template v-for="l in main" :key="l.to || l.title || l.html">
                <button v-if="l.onclick" type="button" @click="l.onclick" class="btn tools-btn" :class="l.class" v-tooltip="l.tooltip" v-text="l.title" v-html="l.html"></button>
                <nuxt-link v-else :to="l.to" class="btn tools-btn" :class="l.class" v-tooltip="l.tooltip" v-text="l.title" v-html="l.html"></nuxt-link>
            </template>
            <template v-if="menu.length">
                <dropdown class="btn tools-btn">
                    <template #toggle="{ show }">
                        <button type="button" class="dropdown-toggle" aria-label="더 보기" :aria-expanded="show"><span class="caret"></span></button>
                    </template>
                    <div class="dropdown-menu dropdown-menu-right">
                        <template v-for="m in menu" :key="m.to || m.title">
                            <button v-if="m.onclick" type="button" @click="m.onclick" class="dropdown-item" :class="m.class">{{ m.title }}</button>
                            <nuxt-link v-else :to="m.to" class="dropdown-item" :class="m.class">{{ m.title }}</nuxt-link>
                        </template>
                    </div>
                </dropdown>
            </template>
        </div>
    </div>
</template>

<script>
import { vTooltip } from 'floating-vue';
import { toast } from "vue-sonner";
import Common from '~/mixins/common';
import dropdown from '../components/dropdown';
import { paths } from '../components/icon';

const svg = (name, cls = '') => `<svg class="risu-icon${cls ? ' ' + cls : ''}" viewBox="0 0 24 24"><path d="${paths[name]}"/></svg>`;

export default {
    directives: { tooltip: vTooltip },
    mixins: [Common],
    components: {
        dropdown
    },
    data() {
        return {
            main: [],
            menu: []
        }
    },
    computed: {
        data() {
            return this.$store.state.page.data;
        },
        adminConvenience() {
            return this.$store.state.localConfig['risu.admin_convenience'] !== false;
        }
    },
    methods: {
        calculate() {
            const uuid = this.data?.uuid;
            this.main = [];
            this.menu = [];
            switch (this.$store.state.page.viewName) {
                case 'wiki':
                    if (this.data.date === null) {
                        this.main.push({
                            to: this.doc_action_link(this.data.document, 'backlink'),
                            title: "역링크"
                        });
                        this.main.push({
                            to: this.doc_action_link(this.data.document, 'acl') + '#namespace.read',
                            title: "ACL"
                        });
                        break;
                    }
                    else if (!uuid) {
                        if (this.data.starred) this.main.push({
                            to: this.doc_action_link(this.data.document, 'member/unstar'),
                            tooltip: "Unstar",
                            html: `${svg('heart', 'star-icon')} <span class="star-count">${this.data.star_count}</span>`
                        });
                        else if (this.data.star_count >= 0) this.main.push({
                            to: this.doc_action_link(this.data.document, 'member/star'),
                            tooltip: "Star",
                            html: `${svg('heart', 'star-icon star-off')} <span class="star-count">${this.data.star_count}</span>`
                        });
                        this.main.push({
                            to: this.doc_action_link(this.data.document, 'backlink'),
                            title: "역링크"
                        });
                        this.main.push({
                            to: this.doc_action_link(this.data.document, 'discuss'),
                            class: this.data.discuss_progress ? 'btn-discuss-progress' : null,
                            title: "토론"
                        });
                        if (this.data.editable === true && this.data.edit_acl_message) this.main.push({
                            onclick: () => this.$emit('onClickEditBtn'),
                            html: `${svg('pencil')} 편집 요청`
                        });
                        else if (this.data.editable === false && this.data.edit_acl_message) this.main.push({
                            onclick: () => this.$emit('onClickEditBtn'),
                            html: `${svg('lock')} 편집`
                        });
                        else this.main.push({
                            to: this.doc_action_link(this.data.document, 'edit'),
                            html: `${svg('pencil')} 편집`
                        });
                        this.main.push({
                            to: this.doc_action_link(this.data.document, 'history', this.data.rev ? { from: this.data.rev } : undefined),
                            title: "역사"
                        });
                        this.main.push({
                            to: this.doc_action_link(this.data.document, 'acl'),
                            title: "ACL"
                        });
                        if (this.data.user) {
                            this.menu.push({
                                to: this.contribution_link(this.data.user.uuid),
                                title: "기여 목록"
                            });
                            if (this.$store.state.session.quick_block && this.adminConvenience) {
                                this.menu.push({
                                    class: 'admin',
                                    onclick: () => this.openQuickACLGroup({
                                        username: this.data.document.title,
                                        note: `${this.doc_fulltitle(this.data.document)} 긴급차단`
                                    }),
                                    title: "사용자 차단"
                                });
                                this.menu.push({
                                    class: 'admin',
                                    to: `/BlockHistory?query=${this.data.user.uuid}&target=text`,
                                    title: "차단 내역"
                                });
                                this.menu.push({
                                    class: 'admin',
                                    onclick: () => this.copyUuid(this.data.document.title, this.data.user.uuid),
                                    title: "UUID 복사"
                                });
                            }
                        }
                        break;
                    }
                case 'raw':
                case 'blame':
                case 'revert':
                case 'diff':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'history', this.data.rev ? { from: this.data.rev } : undefined),
                        class: 'btn-info',
                        title: "역사"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'w', uuid ? { uuid } : undefined),
                        class: this.$store.state.page.viewName === 'wiki' ? 'disabled' : null,
                        title: "보기"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'raw', uuid ? { uuid } : undefined),
                        class: this.$store.state.page.viewName === 'raw' ? 'disabled' : null,
                        title: "RAW"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'blame', uuid ? { uuid } : undefined),
                        class: this.$store.state.page.viewName === 'blame' ? 'disabled' : null,
                        title: "blame"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'revert', uuid ? { uuid } : undefined),
                        class: this.$store.state.page.viewName === 'revert' ? 'disabled' : null,
                        title: "되돌리기"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'diff', uuid ? { uuid } : undefined),
                        class: this.$store.state.page.viewName === 'diff' ? 'disabled' : null,
                        title: "비교"
                    });
                    break;
                case 'notfound':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'backlink'),
                        title: "역링크"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'discuss'),
                        class: this.data.discuss_progress ? 'btn-discuss-progress' : null,
                        title: "토론"
                    });
                    if (this.data.edit_acl_message) this.main.push({
                        onclick: () => this.$emit('onClickEditBtn'),
                        html: `${svg('lock')} 편집`
                    });
                    else this.main.push({
                        to: this.doc_action_link(this.data.document, 'edit'),
                        html: `${svg('pencil')} 편집`
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'history', this.data.rev ? { from: this.data.rev } : undefined),
                        title: "역사"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'acl'),
                        title: "ACL"
                    });
                    break;
                case 'edit':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'move'),
                        title: "이동"
                    });
                    this.main.push({
                        class: "btn-danger",
                        to: this.doc_action_link(this.data.document, 'delete'),
                        title: "삭제"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'history'),
                        title: "역사"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'acl') + '#document.edit',
                        title: "ACL"
                    });
                    break;
                case 'edit_edit_request':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'history'),
                        title: "역사"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'acl') + '#document.edit_request',
                        title: "ACL"
                    });
                    break;
                case 'history':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'discuss'),
                        title: "토론"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'edit'),
                        title: "편집"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'acl'),
                        title: "ACL"
                    });
                    break;
                case 'thread_list':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'edit'),
                        title: "편집"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'acl') + '#document.create_thread',
                        title: "ACL"
                    });
                    break;
                case 'thread_list_close':
                case 'edit_request_close':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'discuss'),
                        title: "토론"
                    });
                    break;
                case 'thread':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'discuss'),
                        title: "토론"
                    });
                    this.main.push({
                        to: this.data.thread?.url ? `/thread/${this.data.thread.url}/acl` : this.doc_action_link(this.data.document, 'acl') + '#document.write_thread_comment',
                        title: "ACL"
                    });
                    break;
                case 'edit_request':
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'discuss'),
                        title: "토론"
                    });
                    this.main.push({
                        to: this.doc_action_link(this.data.document, 'acl') + '#document.edit',
                        title: "ACL"
                    });
                    break;
                case 'contribution':
                case 'contribution_discuss':
                case 'contribution_edit_request':
                    this.main.push({
                        to: this.data.account.type === 1 ? this.doc_action_link(this.user_doc(this.data.account.name), 'w') : '',
                        class: this.data.account.type === 1 ? '' : 'disabled',
                        title: "사용자 문서"
                    });
                    if (this.$store.state.session.quick_block && this.adminConvenience) {
                        if (this.data.account.type !== -1) this.menu.push({
                            class: 'admin',
                            onclick: () => this.openQuickACLGroup({
                                username: this.data.account.type === 1 ? "".concat(this.data.account.name) : undefined,
                                ip: this.data.account.type === 0 ? this.data.account.name + '/' + (this.data.account.name.indexOf('.') === -1 ? "128" : '32') : undefined,
                                note: '기여 목록 긴급차단'
                            }),
                            title: "사용자 차단"
                        });
                        if (this.data.account.type === -1 && this.data.account.uuid) this.menu.push({
                            to: this.doc_action_link('삭제된사용자:' + this.data.account.uuid, 'w'),
                            class: 'admin',
                            title: "삭제된 사용자 문서"
                        });
                        this.menu.push({
                            class: 'admin',
                            to: `/BlockHistory?query=${this.data.account.uuid}&target=text`,
                            title: "차단 내역"
                        });
                        this.menu.push({
                            class: 'admin',
                            onclick: () => this.copyUuid(this.data.account.name, this.data.account.uuid),
                            title: "UUID 복사"
                        });
                    }
                    break;
                case '':
                    if (this.$store.state.page.title === '오류') this.main.push({
                        onclick: () => this.$router.back(),
                        title: "이전 화면"
                    });
                    break;
            }
            if (this.data.menus) this.menu = this.menu.concat(this.data.menus);
        },
        async copyUuid(name, uuid) {
            try {
                await navigator.clipboard.writeText(uuid);
                toast(`${name} 사용자의 UUID가 복사되었습니다.`);
            } catch {
                toast("복사하지 못했습니다.");
            }
        }
    },
    mounted() {
        this.calculate();
    },
    watch: {
        $route() {
            this.$nextTick(() => {
                this.calculate();
            });
        },
        adminConvenience() {
            this.$nextTick(() => {
                this.calculate();
            });
        }
    }
}
</script>
