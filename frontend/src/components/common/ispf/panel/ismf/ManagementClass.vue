<template>
    <div class="master-panel">
        <h3 class="panel-title">MANAGEMENT CLASS APPLICATION SELECTION</h3>
        <div class="panel-content">
            <a-row class="panel-name"
            >To perform Management Class Operations, Specify:
            </a-row
            >
            <a-row>
                <a-col :offset="1">
                    <a-input
                            addonBefore="CDS Name"
                            class="panel-option"
                            v-focus
                            v-model="cdsName"
                    ></a-input>
                </a-col>
            </a-row>
            <a-row>
                <a-col :offset="1">
                    <a-input
                            addonBefore="Management Class Name"
                            class="panel-option"
                            v-model="construct"
                    ></a-input>
                </a-col>
            </a-row>
            <a-row>
                <a-input
                        @pressEnter="onEnter"
                        addonBefore="Select one of the following options  :"
                        class="panel-option"
                        placeholder="Only support option 3"
                ></a-input>
            </a-row>
            <a-row>
                <a-col :offset="1" :span="1" class="panel-cmd">1</a-col>
                <a-col :span="5" class="panel-name">List</a-col>
                <a-col :span="17" class="panel-desc"
                >- Generate a list of Management Classes
                </a-col
                >
            </a-row>
            <a-row>
                <a-col :offset="1" :span="1" class="panel-cmd">2</a-col>
                <a-col :span="5" class="panel-name">Display</a-col>
                <a-col :span="17" class="panel-desc"
                >- Display a Management Classes
                </a-col
                >
            </a-row>
            <a-row>
                <a-col :offset="1" :span="1" class="panel-cmd">3</a-col>
                <a-col :span="5" class="panel-name">Define</a-col>
                <a-col :span="17" class="panel-desc"
                >- Define a Management Classes
                </a-col
                >
            </a-row>
            <a-row>
                <a-col :offset="1" :span="1" class="panel-cmd">4</a-col>
                <a-col :span="5" class="panel-name">Alter</a-col>
                <a-col :span="17" class="panel-desc"
                >- Alter a Management Classes
                </a-col
                >
            </a-row>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                cdsName: "",
                construct: ""
            };
        },
        methods: {
            onEnter(e) {
                if (e.target.value) {
                    this.$store.commit(
                        "ispf/SET_CDS_NAME",
                        this.cdsName.trim().toUpperCase()
                    );
                    this.$store.commit(
                        "ispf/SET_CONSTRUCT",
                        this.construct.trim().toUpperCase()
                    );
                    const panel = `is_3_${e.target.value.trim()}`
                        .replace(/\./g, "_")
                        .toLowerCase();
                    this.$store.commit("ispf/SET_PANEL", panel);
                }
            }
        }
    };
</script>

<style scoped></style>
