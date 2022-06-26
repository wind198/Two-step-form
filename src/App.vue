<script>
import TabFormContainer from "./components/common/TabFormContainer.vue";
import InputField from "./components/common/InputField.vue";
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup

const TAB_LIST = ["tab1", "tab2"];
const TAB_LIST_WITH_PROPS = [
    {
        tabId: "tab1",
        fields: [
            {
                required: true,
                fieldName: "name",
                fieldId: "name",
                fieldType: "text",
            },
            {
                required: true,
                fieldName: "email",
                fieldId: "email",
                fieldType: "email",
            },
        ],
    },
    {
        tabId: "tab2",
        fields: [
            {
                required: true,
                fieldName: "phone",
                fieldId: "phone",
                fieldType: "tel",
            },
            {
                required: true,
                fieldName: "address",
                fieldId: "address",
                fieldType: "text",
            },
        ],
    },
];
const emailRegex = /^[a-zA-Z0-9._-]+@([a-zA-Z_]+\.)+[a-zA-Z]{2,3}$/g;
const telRegex = /^[\+]?[(]?[0-9]{3}[)]?[-\s\.]?[0-9]{3}[-\s\.]?[0-9]{4,6}$/g;

export default {
    data() {
        return {
            tabsActiveStatus: {
                tab1: true,
                tab2: false,
                tab3: false,
            },
            tabSubmitStatus: {
                tab1: false,
                tab2: false,
                tab3: false,
            },
            formData: {
                tab1: {
                    name: {
                        value: "",
                        isValid: true,
                        isDirty: false,
                        message: "",
                    },
                    email: {
                        value: "",
                        isValid: true,
                        isDirty: false,
                        message: "",
                    },
                },
                tab2: {
                    phone: {
                        value: "",
                        isValid: true,
                        isDirty: false,
                        message: "",
                    },
                    address: {
                        value: "",
                        isValid: true,
                        isDirty: false,
                        message: "",
                    },
                },
            },
            isSubmitted: false,
        };
    },
    created() {
        this.tabList = TAB_LIST;
        this.tabListWithProps = TAB_LIST_WITH_PROPS;
    },
    methods: {
        getTabTitle(tabId) {
            switch (tabId) {
                case "tab1":
                    return "Tab One";
                case "tab2":
                    return "Tab Two";
                case "tab3":
                    return "Tab Three";
            }
        },
        changeActiveTab(tabId) {
            const couldNavigate = this.checkIfCouldNavigateFromCurrentTab();

            if (!couldNavigate) return;

            for (const key in this.tabsActiveStatus) {
                this.tabsActiveStatus[key] = false;
            }
            this.tabsActiveStatus[tabId] = true;
        },
        handleInputField(tabId, payload) {
            const { fieldId, value, fieldType } = payload;
            if (
                this.formData[tabId] &&
                this.formData[tabId].hasOwnProperty(fieldId)
            ) {
                this.formData[tabId][fieldId].value = value;
                this.formData[tabId][fieldId].isDirty = true;
                if (this.tabSubmitStatus[tabId]) {
                    this.validateField(tabId, fieldId, value, fieldType);
                }
            }
        },
        handleChangeField(tabId, payload) {
            const { fieldId, value, fieldType } = payload;
            this.validateField(tabId, fieldId, value, fieldType);
        },

        validateField(tabId, fieldId, value, fieldType) {
            if (
                this.formData[tabId] &&
                this.formData[tabId].hasOwnProperty(fieldId)
            ) {
                switch (fieldType) {
                    case "text":
                    case "email":
                    case "tel":
                        let required = false;
                        const targetTab = this.tabListWithProps.find(
                            (t) => t.tabId === tabId
                        );
                        if (targetTab) {
                            const targetField = targetTab.fields.find(
                                (f) => f.fieldId === fieldId
                            );
                            if (targetField) {
                                required = targetField.required;
                                if (required) {
                                    const targetFieldInFormData =
                                        this.formData[tabId][fieldId];
                                    targetFieldInFormData.isValid = !!value;
                                    targetFieldInFormData.message = value
                                        ? ""
                                        : "This field is required";

                                    if (fieldType === "email" && value) {
                                        const match = value.match(emailRegex);
                                        if (!match) {
                                            targetFieldInFormData.message = `"${value}" is not a valid email`;
                                            targetFieldInFormData.isValid = false;
                                        }
                                    }
                                    if (fieldType === "tel" && value) {
                                        const match = value.match(telRegex);
                                        if (!match) {
                                            targetFieldInFormData.message = `"${value}" is not a valid email`;
                                            targetFieldInFormData.isValid = false;
                                        }
                                    }
                                }
                            }
                        }

                    default:
                }
            }
        },
        validateOverallTab(tabId) {
            const fieldListWithinTab = this.formData[tabId];
            let overallTabValidated = true;
            Object.keys(fieldListWithinTab).forEach((fieldId) => {
                const fieldObj = fieldListWithinTab[fieldId];
                const isDirty = fieldObj.isDirty;
                const currentValue = fieldObj.value;
                if (!isDirty) {
                    const targetTabProps = this.tabListWithProps.find(
                        (t) => t.tabId === tabId
                    );
                    if (!targetTabProps) {
                        overallTabValidated = false;
                        return;
                    }
                    const targetFieldProps = targetTabProps.fields.find(
                        (f) => f.fieldId === fieldId
                    );
                    if (!targetFieldProps) {
                        overallTabValidated = false;
                        return;
                    }
                    const { fieldType } = targetFieldProps;
                    this.validateField(tabId, fieldId, currentValue, fieldType);
                }
            });
            return overallTabValidated;
        },
        checkIfCouldNavigateFromCurrentTab() {
            const currentTabId = Object.keys(this.tabsActiveStatus).find(
                (k) => this.tabsActiveStatus[k]
            );
            if (!currentTabId) {
                return false;
            }
            const overallTabValidated = this.validateOverallTab(currentTabId);
            if (!overallTabValidated) {
                return false;
            }
            const couldNavigate = Object.values(
                this.formData[currentTabId]
            ).some((field) => !field.isValid)
                ? false
                : true;
            return couldNavigate;
        },
        navigateFromCurrentTab(tabId, direction) {
            const indexOfCurrentTab = this.tabList.findIndex(
                (t) => t === tabId
            );
            if (indexOfCurrentTab < 0) {
                return;
            }
            let nextIndex = indexOfCurrentTab;
            if (direction === "next") {
                nextIndex =
                    indexOfCurrentTab + 1 > this.tabList.length
                        ? 0
                        : indexOfCurrentTab + 1;
            }
            if (direction === "prev") {
                nextIndex =
                    indexOfCurrentTab - 1 < 0
                        ? this.tabList.length - 1
                        : indexOfCurrentTab - 1;
            }
            this.changeActiveTab(this.tabList[nextIndex]);
        },
        handleClickPrev(e, tabId) {
            this.tabSubmitStatus[tabId] = true;
            const couldNavigate = this.checkIfCouldNavigateFromCurrentTab();
            if (!couldNavigate) {
                return;
            }
            this.navigateFromCurrentTab(tabId, "prev");
        },
        handleClickNext(e, tabId) {
            this.tabSubmitStatus[tabId] = true;

            const couldNavigate = this.checkIfCouldNavigateFromCurrentTab();
            if (!couldNavigate) {
                return;
            }
            this.navigateFromCurrentTab(tabId, "next");
        },
        handleClickSubmit(e, tabId) {
            this.tabSubmitStatus[tabId] = true;
            const couldNavigate = this.checkIfCouldNavigateFromCurrentTab();
            if (!couldNavigate) {
                return;
            }
            alert(JSON.stringify(this.formData));
        },
    },
    components: { TabFormContainer, InputField },
};
</script>

<template>
    <div class="app-container">
        <nav class="nav">
            <div
                v-for="tabId in tabList"
                key="tab"
                :class="[tabsActiveStatus[tabId] ? 'active' : '']"
                @click="changeActiveTab(tabId)"
            >
                {{ getTabTitle(tabId) }}
            </div>
        </nav>
        <main class="main">
            <div
                v-for="tab in tabListWithProps"
                :class="[
                    'tab-container',
                    !tabsActiveStatus[tab.tabId] ? 'inactive' : 'active',
                ]"
                :key="tab.tabId"
            >
                <TabFormContainer v-if="tabsActiveStatus[tab.tabId]">
                    <template #form__content>
                        <div
                            v-for="field in tab.fields"
                            :key="field.fieldId"
                            :class="[
                                'form-control',
                                formData[tab.tabId][field.fieldId].isValid
                                    ? 'success'
                                    : 'error',
                            ]"
                        >
                            <label :for="field.fieldId">{{
                                field.fieldName
                            }}</label>
                            <InputField
                                :fieldName="field.fieldName"
                                :fieldId="field.fieldId"
                                :fieldType="field.fieldType"
                                :fieldValue="
                                    formData[tab.tabId][field.fieldId].value
                                "
                                @input-field="
                                    (payload) =>
                                        handleInputField(tab.tabId, payload)
                                "
                                @change-field="
                                    (payload) =>
                                        handleChangeField(tab.tabId, payload)
                                "
                            />
                            <p
                                :class="[
                                    'message',
                                    formData[tab.tabId][field.fieldId].isValid
                                        ? 'success'
                                        : 'error',
                                ]"
                            >
                                {{ formData[tab.tabId][field.fieldId].message }}
                            </p>
                        </div>
                    </template>
                    <template #form__actions>
                        <button
                            v-if="tab.tabId === 'tab2'"
                            type="button"
                            @click="handleClickPrev($event, tab.tabId)"
                            class="form__button prev"
                        >
                            Previous
                        </button>
                        <button
                            v-if="tab.tabId === 'tab2'"
                            type="button"
                            @click="handleClickSubmit($event, tab.tabId)"
                            class="form__button submit"
                        >
                            Submit
                        </button>
                        <button
                            v-if="tab.tabId === 'tab1'"
                            type="button"
                            @click="handleClickNext($event, tab.tabId)"
                            class="form__button next"
                        >
                            Next
                        </button>
                    </template>
                </TabFormContainer>
            </div>
        </main>
    </div>
</template>

<style lang="scss">
.app-container {
    width: 60vw;
    min-height: 50vh;
    margin: auto;
    margin-top: 60px;
    background-color: #fafafa;
    font-family: sans-serif;
    font-size: 14px;
    padding: 8px;
    border-radius: 16px;
    box-shadow: 0 4px 4px rgba($color: #000000, $alpha: 0.2);
    display: flex;
    flex-direction: column;
}
.nav {
    display: flex;
    justify-content: space-between;
    padding: 6px 32px;
    background-color: rgb(116, 170, 250);
    border-radius: 12px;

    > *:not(style) {
        flex: 1;
        padding: 6px 0;
        border-radius: 12px;
        font-size: 20px;
        font-weight: 500;
        text-align: center;
        color: white;
        cursor: pointer;
        &.active {
            background-color: white;
            color: #000;
        }
    }
}
.main {
    height: 100%;
    flex: 1;
    display: flex;
    flex-direction: column;
}
.tab-container {
    display: flex;
    &.active {
        flex: 1;
    }
    &.inactive {
        height: 0;
    }
}
.form-control {
    display: flex;
    flex-direction: column;
    label {
        display: block;
        margin-bottom: 4px;
        text-transform: capitalize;
    }
    .message {
        font-size: 12px;
    }
    &.error {
        > label,
        > p {
            color: red;
        }
        > input {
            outline: 1px solid red;
        }
    }
}
.form__button {
    min-height: 32px;
    min-width: 80px;
    font-size: 14;
    font-weight: 600;
    border: none;
    cursor: pointer;
    border-radius: 4px;
    box-shadow: 0 1px 3px rgba($color: #000000, $alpha: 0.2);
    &:hover {
        box-shadow: 0 1px 2px rgba($color: #000000, $alpha: 0.3),
            0 1px 6px rgba($color: #000000, $alpha: 0.1);
    }
    &.next {
        background-color: rgb(30, 100, 252);
        color: #fff;
    }
    &.submit {
        background-color: rgb(254 164 0);
        color: #fff;
    }
}
</style>
