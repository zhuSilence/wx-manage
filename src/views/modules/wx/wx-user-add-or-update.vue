<template>
    <el-dialog title="额度设置" :close-on-click-modal="false" :visible.sync="visible">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="100px" size="mini">
            <el-form-item label="用户 OpenId" prop="id">
                <el-input v-model="dataForm.id" disabled placeholder="OpenId"></el-input>
            </el-form-item>
            <el-form-item label="输入额度" prop="url">
                <el-input-number v-model="dataForm.count" placeholder="数字类型"></el-input-number>
            </el-form-item>

        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
        </span>
    </el-dialog>
</template>

<script>
export default {
    data() {
        return {
            visible: false,
            dataForm: {
                id: 0,
                count: 0
            },
            dataRule: {
                count: [
                    { required: true, message: '额度不能为空', trigger: 'blur' }
                ]
            }
        }
    },
    methods: {
        init(id) {
            console.log('init',id)
            this.dataForm.id = id || 0
            this.visible = true
            this.$nextTick(() => {
                this.$refs['dataForm'].resetFields()
                if (this.dataForm.id) {
                    this.$http({
                        url: this.$http.adornUrl(`/manage/wxUser/info/${this.dataForm.id}`),
                        method: 'get',
                        params: this.$http.adornParams()
                    }).then(({ data }) => {
                        if (data && data.code === 200) {
                            this.dataForm.id = data.wxUser.openid;
                            let extraInfo = JSON.parse(data.wxUser.extraInfo);
                            this.dataForm.count = extraInfo.openApiCount;
                        }else{
                            this.$message.error(data.msg)
                        }
                    })
                }
            })
        },

        // 表单提交
        dataFormSubmit() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl('/manage/wxUser/update'),
                        method: 'post',
                        data: this.$http.adornData(this.dataForm)
                    }).then(({ data }) => {
                        if (data && data.code === 200) {
                            this.$message({
                                message: '操作成功',
                                type: 'success',
                                duration: 1500,
                                onClose: () => {
                                    this.visible = false
                                    this.$emit('refreshDataList')
                                }
                            })
                        } else {
                            this.$message.error(data.msg)
                        }
                    })
                }
            })
        }
    }
}
</script>