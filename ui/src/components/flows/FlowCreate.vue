<template>
    <div>
        <editor-view
            :flow-id="defaultFlowTemplate.id"
            :namespace="defaultFlowTemplate.namespace"
            :is-creating="true"
            :flow-graph="flowGraph"
            :is-read-only="false"
            :total="total"
            :guided-properties="guidedProperties"
            :flow-error="flowError"
            :flow-deprecations="flowDeprecations"
            :flow="sourceWrapper"
        />
    </div>
    <div id="guided-right" />
</template>

<script>
    import EditorView from "../inputs/EditorView.vue";
    import {mapGetters, mapState} from "vuex";
    import RouteContext from "../../mixins/routeContext";

    export default {
        mixins: [RouteContext],
        components: {
            EditorView
        },
        beforeUnmount() {
            this.$store.commit("flow/setFlowError", undefined);
            this.$store.commit("flow/setFlowDeprecations", undefined);
        },
        computed: {
            sourceWrapper() {
                return {source: this.defaultFlowTemplate};
            },
            defaultFlowTemplate() {
                if(this.$route.query.copy && this.flow){
                    return this.flow.source;
                }

                return `id: incentive-template
namespace: dev
tasks:
  - id: hello
    type: io.kestra.core.tasks.log.Log
    message: Zappyride team wishes you a great day! 👋`;
            },
            ...mapState("flow", ["flowGraph", "total"]),
            ...mapState("auth", ["user"]),
            ...mapState("plugin", ["pluginSingleList", "pluginsDocumentation"]),
            ...mapGetters("core", ["guidedProperties"]),
            ...mapGetters("flow", ["flow", "flowError", "flowDeprecations"]),
            routeInfo() {
                return {
                    title: this.$t("flows")
                };
            },
        },
        beforeRouteLeave(to, from, next) {
            this.$store.commit("flow/setFlow", null);
            next();
        }
    };
</script>
