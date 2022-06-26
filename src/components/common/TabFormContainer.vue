<script>
export default {
    methods: {
        handleSubmit(e) {
            e.preventDefault();
        },
        handleReset(e) {
            e.preventDefault();
        },
    },
};
</script>

<template>
    <form
        class="tab__form"
        @submit="handleSubmit($event)"
        @reset="handleReset($event)"
    >
        <div class="form__content">
            <slot name="form__content"></slot>
        </div>
        <div class="form__actions">
            <slot name="form__actions"></slot>
        </div>
    </form>
</template>

<style lang="scss">
.tab__form {
    width: 100%;
    flex: 1;
    display: flex;
    flex-direction: column;

    .form__content {
        flex: 1;
        padding: 8px;
        display: flex;
        flex-direction: column;
        > *:not(style) {
            margin-bottom: 16px;
            &::last-child {
                margin-bottom: 0;
            }
        }
    }
    .form__actions {
        padding: 12px 8px;
        display: flex;
        justify-content: flex-end;
        & > *:not(style) {
            margin-right: 8px;
            &:last-child {
                margin-right: 0;
            }
        }
    }
}
</style>
