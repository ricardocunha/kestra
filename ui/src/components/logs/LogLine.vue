<template>
    <div class="line font-monospace" v-if="filtered">
        <span :class="levelClass" class="header-badge log-level el-tag noselect">{{ log.level.padEnd(9) }}</span>
        <span class="header-badge noselect">
            {{ $filters.date(log.timestamp, "iso") }}
        </span>
        <span v-for="(meta, x) in metaWithValue" :key="x">
            <span class="header-badge property">
                <span>{{ meta.key }}</span>
                <template v-if="meta.router">
                    <router-link :to="meta.router">{{ meta.value }}</router-link>
                </template>
                <template v-else>
                    {{ meta.value }}
                </template>
            </span>
        </span>
        <span class="message" v-html="message" />
    </div>
</template>
<script>
    import Convert from "ansi-to-html"
    import xss from "xss";
    let convert = new Convert();

    export default {
        props: {
            log: {
                type: Object,
                required: true,
            },
            filter: {
                type: String,
                default: "",
            },
            level: {
                type: String,
                required: true,
            },
            excludeMetas: {
                type: Array,
                default: () => [],
            },
        },
        computed: {
            metaWithValue() {
                const metaWithValue = [];
                const excludes = [
                    "message",
                    "timestamp",
                    "thread",
                    "taskRunId",
                    "level",
                    "index"
                ];
                excludes.push.apply(excludes, this.excludeMetas);
                for (const key in this.log) {
                    if (this.log[key] && !excludes.includes(key)) {
                        let meta = {key, value: this.log[key]};
                        if (key === "executionId") {
                            meta["router"] = {name: "executions/update", params: {
                                namespace: this.log["namespace"],
                                flowId: this.log["flowId"],
                                id: this.log[key]
                            }};
                        }

                        if (key === "namespace") {
                            meta["router"] = {name: "flows/list", query: {namespace: this.log[key]}};
                        }


                        if (key === "flowId") {
                            meta["router"] = {name: "flows/update", params: {namespace: this.log["namespace"], id: this.log[key]}};
                        }

                        if (key === "attemptNumber") {
                            meta.value = meta.value + 1;
                        }

                        metaWithValue.push(meta);
                    }
                }
                return metaWithValue;
            },
            levelClass() {
                return {
                    TRACE: "",
                    DEBUG: "el-tag--info",
                    INFO: "el-tag--success",
                    WARN: "el-tag--warning",
                    ERROR: "el-tag--danger",
                }[this.log.level];
            },
            filtered() {
                return (
                    this.filter === "" || (
                        this.log.message &&
                        this.log.message.toLowerCase().includes(this.filter)
                    )
                );
            },
            message() {
                return !this.log.message ? "" : convert.toHtml(xss(this.log.message));
            }
        },
    };
</script>
<style scoped lang="scss">
    div.line {
        white-space: pre-wrap;
        word-break: break-all;
        padding: 2px 2px 0 2px;

        span {
            margin-bottom: 2px;
        }
        .el-tag {
            border-radius: 0;
            border: 0;
            height: auto;
        }

        .header-badge {
            display: inline-block;
            font-size: 95%;
            padding: calc(var(--spacer) / 3) calc(var(--spacer) / 3);
            line-height: 1;
            text-align: center;
            white-space: nowrap;
            vertical-align: baseline;
            margin-right: calc(var(--spacer) / 3);
            &:not(.el-tag) {
                background-color: var(--bs-gray-300);
            }

            span:first-child {
                margin-right: 6px;
                font-family: var(--bs-font-sans-serif);
                color: var(--bs-gray-500);

                html.dark & {
                    color: var(--bs-gray-700);
                }
                user-select: none;

                &::after{
                    content: ":";
                }
            }

            &:not(.el-tag), & a {
                color: var(--bs-gray-700);
                border-radius: var(--bs-border-radius);
            }

            & a:hover {
                color: var(--bs-link-color);
            }

            &.log-level {
                white-space: pre;
                border-radius: var(--bs-border-radius);
            }
        }

        .message {
            padding: 0 calc(var(--spacer) / 2);
        }

        .noselect {
            user-select: none;
        }

    }
</style>
