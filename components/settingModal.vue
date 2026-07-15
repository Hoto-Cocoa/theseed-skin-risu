<template>
    <setting>
        <setting-item-select label="사이드바 설정" ckey="risu.sidebar" :default="sidebar">
            <option value="right">우측 표시</option>
            <option value="hide">숨김</option>
            <option value="footer">하단 표시</option>
        </setting-item-select>
        <setting-item-checkbox label="내비게이션 바 고정" ckey="risu.fixed_navbar" :default="true" />
        <setting-item-checkbox label="페이지 이동 시 검색 창 초기화" ckey="risu.reset_search_on_move" :default="true" />
        <setting-item-checkbox label="넓은 화면 모드" ckey="risu.wide_mode" />
        <template v-if="$store.state.session.quick_block">
            <hr />
            <setting-item-checkbox label="[ADMIN] 관리 편의성 개선" ckey="risu.admin_convenience" :default="true" />
        </template>
    </setting>
</template>

<script>
import Common from '~/mixins/common';

import Setting from '~/components/setting';
import SettingItemCheckbox from '~/components/settingItemCheckbox';
import SettingItemSelect from '~/components/settingItemSelect';

import { isMobile } from '~/utils';

export default {
    mixins: [Common],
    components: {
        Setting,
        SettingItemCheckbox,
        SettingItemSelect,
    },
    computed: {
        sidebar() {
            return isMobile ? "hide" : "right";
        }
    }
}
</script>
