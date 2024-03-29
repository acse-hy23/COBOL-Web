<template>
    <div>
        <h1>我的实验</h1>
        <a-spin :spinning="isLoading" size="large">
            <a-icon :spin="true" slot="indicator" type="loading-3-quarters"/>
            <a-table :columns="columns" :dataSource="data" rowKey="name">
        <span slot="status" slot-scope="{ status }">
          <a-badge :status="status | statusFilter"/>
          {{ status | statusFilter2 }}
        </span>
                <span slot="action" slot-scope="record">
          <a :disabled="!record.url" :href="record.url" target="_blank">
            <a-icon type="eye"/> 浏览
          </a>
          <a-divider type="vertical"/>
          <a
                  :disabled="!record.url"
                  :download="record.disposition"
                  :href="record.url"
          >
            <a-icon type="download"/> 下载
          </a>
          <a-divider type="vertical"/>
          <a
                  :disabled="record.status !== 'saved'"
                  @click="handelSubmitClick(record)"
          >
            <a-icon type="upload"/> 提交
          </a>
        </span>
            </a-table>
        </a-spin>
        <a-modal
                :confirmLoading="isSubmitting"
                @ok="submitLabReport"
                title="朋友，确定提交么？"
                v-model="visible"
        >
            <p>
        <span style="color: coral">
          重要：请确保{{ clickedRecord.name }}实验的所有步骤的回答都已经保存。
        </span>
                <a-popover>
                    <template slot="content">
                        在实验界面点击"保存本页"按钮即可保存该步骤的回答。
                    </template>
                    <a-icon style="cursor: pointer" type="question-circle"/>
                </a-popover>
            </p>
            <p>
                注意：提交实验报告后老师便可以看到并批改，同时你将<span
                    style="color: coral"
            >无法更改</span
            >你的回答。
            </p>
            <p>
                建议在正式提交之前预览一下实验报告 😁。<a
                    :href="clickedRecord.url"
                    target="_blank"
            >[ 点我预览 ]</a
            >
            </p>
            <p>确定提交报告么？</p>
        </a-modal>
    </div>
</template>

<script>
    import Axios from "axios";

    const columns = [
        {title: "实验名称", dataIndex: "name"},
        {title: "实验状态", key: "status", scopedSlots: {customRender: "status"}},
        {title: "评分", dataIndex: "score"},
        {title: "评语", dataIndex: "comment", width: "300px"},
        {
            title: "操作",
            key: "action",
            scopedSlots: {customRender: "action"}
        }
    ];

    const data = [
        {
            name: "COBOL子程序",
            labId: "Subroutine",
            status: "",
            score: "",
            comment: ""
        }
    ];

    export default {
        data() {
            return {
                columns,
                data,
                allRates: [],
                visible: false,
                clickedRecord: {},
                isLoading: true,
                isSubmitting: false
            };
        },
        created() {
            this.init();
        },
        methods: {
            // init page data
            async init() {
                this.isLoading = true;
                // 获取提交状态
                let labStatus = await this.$http.get("/api/db/getLabStatus");
                this.data.forEach((item, index, arr) => {
                    arr[index].status = labStatus.body.find(
                        lab => lab.lab === item.labId
                    ).status;
                });

                // 获取分数（若已经批改）
                this.allRates = await this.$http.get("/api/db/checkScore");

                this.data.forEach(lab => {
                    this.$http
                        .get("/api/db/getReports", {
                            params: {
                                lab: lab.labId
                            },
                            responseType: "arraybuffer"
                        })
                        .then(res => {
                            if (res.data.byteLength) {
                                // 生成 pdf url
                                // lab.status = 1; // 已提交
                                lab.disposition = res.headers.map["content-disposition"][0]
                                    .split("=")[1]
                                    .replace(/"/g, "");
                                let binaryData = [];
                                binaryData.push(res.body);

                                lab.url = window.URL.createObjectURL(
                                    new Blob(binaryData, {
                                        type: "application/pdf"
                                    })
                                );

                                this.data = [...this.data]; // 更新数据

                                // 是否批改
                                if (this.allRates && +this.allRates.status === 200) {
                                    let ratedLabIndex = this.allRates.body.findIndex(rate => {
                                        return rate.lab.toUpperCase() === lab.labId.toUpperCase();
                                    });
                                    if (ratedLabIndex !== -1) {
                                        lab.comment = this.allRates.body[ratedLabIndex].comment;
                                        lab.score = this.allRates.body[ratedLabIndex].score;
                                        lab.status = "scored";
                                    }
                                }
                            }
                        });
                });

                this.isLoading = false;
            },
            handelSubmitClick(record) {
                this.visible = true;
                this.clickedRecord = record;
            },
            // submit an entire lab e.g. RACF
            submitLabReport() {
                this.isSubmitting = true;
                Axios.post("/api/db/submitLab", {
                    lab: this.clickedRecord.labId
                })
                    .then(() => {
                        this.$message
                            .success(
                                `✨成功提交${this.clickedRecord.name}实验报告，等待老师批阅`
                            )
                            .then();
                    })
                    .catch(e => {
                        this.$message.error("提交失败：" + e.message).then();
                    })
                    .finally(() => {
                        this.isSubmitting = false;
                        this.visible = false;
                        this.init();
                    });
            }
        },
        filters: {
            statusFilter(status) {
                switch (status) {
                    case "unsaved":
                        return "default"; // 未保存
                    case "saved":
                        return "warning"; // 已保存
                    case "submitted":
                        return "processing"; // 已提交
                    case "scored":
                        return "success"; // 已批改
                }
            },
            statusFilter2(status) {
                switch (status) {
                    case "unsaved":
                        return "未保存";
                    case "saved":
                        return "已保存";
                    case "submitted":
                        return "已提交";
                    case "scored":
                        return "已批改";
                    default:
                        return "获取中...";
                }
            }
        }
    };
</script>
