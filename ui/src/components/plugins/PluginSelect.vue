<template>
    <el-select
        :model-value="modelValue"
        placeholder=""
        filterable
        :persistent="false"
        @update:model-value="onInput"
    >
        <el-option
            v-for="item in taskModels.sort()"
            :key="item"
            :label="item"
            :value="item"
        >
            <span class="options">
                <task-icon :cls="item" :only-icon="true" />
                <span>
                    {{ item }}
                </span>
            </span>
        </el-option>

        <template #prefix>
            <task-icon v-if="modelValue" :cls="modelValue" :only-icon="true" />
        </template>
    </el-select>
</template>

<script>
    import {mapState} from "vuex";
    import TaskIcon from "@kestra-io/ui-libs/src/components/misc/TaskIcon.vue";

    export default {
        components: {
            TaskIcon
        },
        props: {
            modelValue: {
                type: String,
                required: false,
                default: undefined,
            },
            section: {
                type: String,
                required: false,
                default: undefined,
            },
        },
        emits: ["update:modelValue"],
        created() {
            this.$store.dispatch("plugin/list");
        },
        computed: {
            ...mapState("plugin", ["plugin", "plugins"]),
            taskModels() {
                const taskModels = [];
                for (const plugin of this.plugins || []) {
                    taskModels.push.apply(taskModels, plugin[this.section.toLowerCase()]);
                }
                return taskModels;
            },
        },
        methods: {
            onInput(value) {
                this.$emit("update:modelValue", value);
            },
        },
    };
</script>

<style lang="scss" scoped>
    :deep(div.wrapper) {
        display: inline-block;
        width: 20px;
        height: 20px;
        margin-right: var(--spacer);
    }

    :deep(.el-input__prefix-inner) {
        .wrapper {
            top: 0;
            margin-right: 0;
        }
    }
</style>
