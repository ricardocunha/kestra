<template>
    <div>
        <collapse>
            <el-form-item>
                <el-input
                    v-model="filter"
                    @update:model-value="onChange"
                    :placeholder="$t('search')"
                >
                    <template #suffix>
                        <magnify />
                    </template>
                </el-input>
            </el-form-item>
            <el-form-item>
                <log-level-selector
                    v-model="level"
                    @update:model-value="onChange"
                />
            </el-form-item>
            <el-form-item>
                <el-button @click="handleLogDisplay()">
                    {{ logDisplayButtonText }}
                </el-button>
            </el-form-item>
            <el-form-item>
                <el-button-group>
                    <el-button @click="downloadContent()">
                        <kicon :tooltip="$t('download logs')">
                            <download />
                        </kicon>
                    </el-button>
                </el-button-group>
            </el-form-item>
        </collapse>

        <log-list
            v-if="execution.state.current === State.RUNNING"
            :level="level"
            :exclude-metas="['namespace', 'flowId', 'taskId', 'executionId']"
            :filter="filter"
            @follow="forwardEvent('follow', $event)"
            @opened-taskruns-count="openedTaskrunsCounts['running'] = $event"
            :logs-to-open-parent="logsToOpen"
        />
        <div v-else-if="execution.state.current !== State.RUNNING">
            <log-list
                v-for="taskRun in taskRunList"
                :key="taskRun.id"
                :task-run-id="taskRun.id"
                :attempt-number="taskRun.attempt"
                :level="level"
                :exclude-metas="['namespace', 'flowId', 'taskId', 'executionId']"
                :filter="filter"
                @follow="forwardEvent('follow', $event)"
                @opened-taskruns-count="openedTaskrunsCounts[taskRun.id] = $event"
                :logs-to-open-parent="logsToOpen"
            />
        </div>
    </div>
</template>

<script>
    import LogList from "../logs/LogList.vue";
    import {mapState} from "vuex";
    import Download from "vue-material-design-icons/Download.vue";
    import Magnify from "vue-material-design-icons/Magnify.vue";
    import Kicon from "../Kicon.vue";
    import LogLevelSelector from "../logs/LogLevelSelector.vue";
    import Collapse from "../layout/Collapse.vue";
    import State from "../../utils/state";

    export default {
        components: {
            LogList,
            LogLevelSelector,
            Kicon,
            Download,
            Magnify,
            Collapse
        },
        data() {
            return {
                fullscreen: false,
                level: undefined,
                filter: undefined,
                logsToOpen: undefined,
                openedTaskrunsCounts: {}
            };
        },
        created() {
            this.level = (this.$route.query.level || localStorage.getItem("defaultLogLevel") || "INFO");
            this.filter = (this.$route.query.q || undefined);
        },
        computed: {
            State() {
                return State
            },
            ...mapState("execution", ["execution", "taskRun", "logs"]),
            downloadName() {
                return `kestra-execution-${this.$moment().format("YYYYMMDDHHmmss")}-${this.execution.id}.log`
            },
            taskRunList() {
                const fullList = [];
                for (const taskRun of (this.execution.taskRunList || [])) {
                    for (const attempt in (taskRun.attempts ?? [{}])) {
                        fullList.push({
                            ...taskRun,
                            attempt: parseInt(attempt),
                        })
                    }
                }
                return fullList
            },
            openedTaskrunsTotal() {
                return Object.values(this.openedTaskrunsCounts).reduce((prev, count) => prev + count, 0);
            },
            logDisplayButtonText() {
                return this.openedTaskrunsTotal === 0 ? this.$t("expand all") : this.$t("collapse all")
            }
        },
        methods: {
            downloadContent() {
                this.$store.dispatch("execution/downloadLogs", {
                    executionId: this.execution.id,
                    params: {
                        minLevel: this.level
                    }
                }).then((response) => {
                    const url = window.URL.createObjectURL(new Blob([response]));
                    const link = document.createElement("a");
                    link.href = url;
                    link.setAttribute("download", this.downloadName);
                    document.body.appendChild(link);
                    link.click();
                });
            },
            forwardEvent(type, event) {
                this.$emit(type, event);
            },
            prevent(event) {
                event.preventDefault();
            },
            onChange() {
                this.$router.push({query: {...this.$route.query, q: this.filter, level: this.level, page: 1}});
            },
            handleLogDisplay() {
                if(this.openedTaskrunsTotal === 0) {
                    this.logsToOpen = State.arrayAllStates().map(s => s.name)
                } else {
                    this.logsToOpen = []
                }
            }
        }
    };
</script>
