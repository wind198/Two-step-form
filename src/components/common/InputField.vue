<script>
export default {
    props: {
        fieldName: String,
        fieldId: String,
        fieldType: String,
        fieldValue: String,
    },
    emits: ["input-field", "change-field"],
    methods: {
        handleInput($event) {
            this.$emit("input-field", {
                fieldId: this.fieldId,
                value: $event.target.value,
                fieldType: this.fieldType,
            });
        },
        handleChange($event) {
            this.$emit("change-field", {
                fieldId: this.fieldId,
                value: $event.target.value,
                fieldType: this.fieldType,
            });
        },
    },
};
</script>

<template>
    <input
        :type="fieldType"
        :id="fieldId"
        :name="fieldName"
        :value="fieldValue"
        @input="handleInput"
        @change="handleChange"
    />
</template>

<style lang="scss" scoped>
input[type="text"],
input[type="email"],
input[type="tel"] {
    display: block;
    padding: 8px 12px;
    font-size: 14px;
    border: none;
    border-radius: 4px;
    box-shadow: 0 0px 1px rgba($color: #000000, $alpha: 0.2);
    &:focus-visible {
        outline: 1px solid rgb(75, 122, 250);
    }
}
</style>
