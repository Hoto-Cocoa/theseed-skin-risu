<template>
    <svg class="risu-icon" viewBox="0 0 24 24" aria-hidden="true"><path :d="path" /></svg>
</template>

<script>
export const paths = {
    refresh: 'M12 5V2L7 6l5 4V7a5 5 0 1 1-4.9 6H5a7 7 0 1 0 7-8z',
    comments: 'M2 5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v10a2 2 0 0 1-2 2H9l-5 4v-4a2 2 0 0 1-2-2V5z',
    sparkle: 'M12 2l2.2 5.8L20 10l-5.8 2.2L12 18l-2.2-5.8L4 10l5.8-2.2L12 2z',
    sliders: 'M3 6h18v2H3zM12 3.5h3v7h-3zM3 16h18v2H3zM7 13.5h3v7H7z',
    search: 'M10 4a6 6 0 1 1 0 12 6 6 0 0 1 0-12zm0-2a8 8 0 1 0 4.9 14.3l4.4 4.4a1 1 0 0 0 1.4-1.4l-4.4-4.4A8 8 0 0 0 10 2z',
    'arrow-right': 'M13.3 5.3a1 1 0 0 1 1.4 0l6 6a1 1 0 0 1 0 1.4l-6 6a1 1 0 0 1-1.4-1.4l4.3-4.3H4a1 1 0 1 1 0-2h13.6l-4.3-4.3a1 1 0 0 1 0-1.4z',
    user: 'M12 12a5 5 0 1 0-5-5 5 5 0 0 0 5 5zm0 2c-4.4 0-8 2.2-8 5v3h16v-3c0-2.8-3.6-5-8-5z',
    pencil: 'M3 17.3V21h3.7L17.8 9.9l-3.7-3.7L3 17.3zM20.7 7a1 1 0 0 0 0-1.4l-2.3-2.3a1 1 0 0 0-1.4 0l-1.8 1.8 3.7 3.7L20.7 7z',
    lock: 'M12 2a5 5 0 0 0-5 5v3H6a2 2 0 0 0-2 2v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8a2 2 0 0 0-2-2h-1V7a5 5 0 0 0-5-5zm-3 8V7a3 3 0 0 1 6 0v3H9z',
    heart: 'M12 21s-7.5-4.7-10-9.3C.6 8.4 2.6 5 6 5c2.2 0 3.7 1.2 4.5 2.6L12 9l1.5-1.4C14.3 6.2 15.8 5 18 5c3.4 0 5.4 3.4 4 6.7-2.5 4.6-10 9.3-10 9.3z',
    list: 'M4 5h2v2H4zM8 5h12v2H8zM4 11h2v2H4zM8 11h12v2H8zM4 17h2v2H4zM8 17h12v2H8z',
    'arrow-up': 'M12 4l7 7-1.4 1.4L13 7.8V20h-2V7.8l-4.6 4.6L5 11l7-7z',
    'arrow-down': 'M12 20l-7-7 1.4-1.4 4.6 4.6V4h2v12.2l4.6-4.6L19 13l-7 7z',
    bell: 'M12 2a6 6 0 0 0-6 6v3.3l-1.7 3.4a1 1 0 0 0 .9 1.5h13.6a1 1 0 0 0 .9-1.5L18 11.3V8a6 6 0 0 0-6-6zM9.8 18a2.3 2.3 0 0 0 4.4 0H9.8z'
};

export default {
    props: {
        name: {
            type: String,
            required: true
        }
    },
    computed: {
        path() {
            return paths[this.name] ?? '';
        }
    }
}
</script>
