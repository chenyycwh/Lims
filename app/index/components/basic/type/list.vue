<template>
    <div>
        <h1 class="page-title"> 监测类别列表
            <small>／Type</small>
        </h1>
        <div class="row">
            <div class="col-md-12">
                <!-- BEGIN BORDERED TABLE PORTLET-->
                <div class="portlet light portlet-fit bordered">
                    <div class="portlet-title">
                        <div class="caption">
                            <div class="clearfix">
                                <a href="javascript:;" @click="create" class="btn btn-sm green"> 创 建
                                    <i class="fa fa-plus"></i>
                                </a>
                                <a href="javascript:;" class="btn btn-sm btn-info" id="selectChange"> 选 择
                                    <i class="fa fa-check-square-o"></i>
                                </a>
                                <a href="javascript:;" @click="removeAll" class="btn btn-sm red"> 删 除
                                    <i class="fa fa-trash-o"></i>
                                </a>
                                <a href="javascript:;" @click="total" class="btn btn-sm default"> 全 部
                                    <i class="fa fa-list"></i>
                                </a>
                            </div>
                        </div>
                        <div class="actions">
                            <div class="input-icon right">
                                <i class="fa fa-search"></i>
                                <input type="text" class="form-control" placeholder="搜索..."
                                       @keyup.enter="search($event)">
                            </div>
                        </div>
                    </div>
                    <div>
                        <div class="table-scrollable table-scrollable-borderless">
                            <table class="table table-hover table-light">
                                <thead>
                                <tr class="uppercase">
                                    <th> 选择</th>
                                    <th> 编号</th>
                                    <th> 监测类别名称</th>
                                    <th> 样品标识符</th>
                                    <th> 操作</th>
                                </tr>
                                </thead>
                                <tbody>
                                <template v-for="(item,index) in typeList">
                                    <tr>
                                        <td class="text-center">
                                            <label class="mt-checkbox mt-checkbox-outline">
                                                <input type="checkbox" :value="item.id" name="select"
                                                       v-model="selected">
                                                <span></span>
                                            </label>
                                        </td>
                                        <td class="text-center"> {{index+1}}</td>
                                        <td class="text-center"> {{item.name}}</td>
                                        <td class="text-center"> {{item.identifier}}</td>
                                        <td class="text-center">
                                            <button type="button" class="btn btn-sm blue btn-outline"
                                                    @click="edit(item)">修 改
                                            </button>
                                            <button type="button" class="btn btn-sm red btn-outline"
                                                    @click="remove(item.id)">删 除
                                            </button>
                                        </td>
                                    </tr>
                                </template>
                                </tbody>
                            </table>
                            <!-- Pagination -->
                            <div class="pagination pull-right">
                                <div class="M-box front pull-right" style="margin-top:10px; "></div>
                            </div>
                            <!-- End Pagination -->
                            <div class="clearfix"></div>
                        </div>
                    </div>
                </div>
                <!-- END BORDERED TABLE PORTLET-->
            </div>

        </div>
</template>

<script>
    module.exports = {
        data(){
            return {
                typeList: [],
                currentPage: 1,
                condition: "",
                selected: []
            }
        },
        mounted(){
            var me = this;
            me.getData();
            BlogUtils.selectAll("select", jQuery("#selectChange"));
        },
        methods: {
            fetchData (pageNum, rowCount) {
                var me = this;
                this.$http.get('/api/type/list', {
                    params: {
                        rowCount: rowCount,
                        currentPage: pageNum,
                        condition: this.condition
                    }
                }).then((response) => {
                    var data = response.data;
                    me.typeList = data.results;
                }, (response) => {
                    serverErrorInfo(response);
                });
            },
            //渲染页码
            fetchPages (rowCount) {
                var me = this;
                this.$http.get('/api/type/list', {
                    params: {
                        rowCount: rowCount,
                        currentPage: 1,
                        condition: me.condition
                    }
                }).then((response) => {
                    var data = response.data;
                    jQuery(".M-box").pagination({
                        pageCount: data.totalPage || 1,
                        coping: true,
                        homePage: '首页',
                        endPage: '末页',
                        prevContent: '上页',
                        nextContent: '下页',
                        callback: function (data) {
                            me.fetchData(data.getCurrent(), rowCount, me.condition);
                            me.currentPage = data.getCurrent();
                        }
                    });
                }, (response) => {
                    serverErrorInfo(response);
                });
            },
            search(e){
                var me = this;
                var value = e.target.value;
                me.currentPag = 1;
                me.condition = value ? "name=" + encodeURI(value) : "";
                me.getData();
            },
            create(){
                router.push("/type/create");
            },
            remove(id){
                var me = this;
                confirm({
                    content: "是否删除当前监测类别信息？",
                    success: function () {
                        me.$http.get("/api/type/delete", {
                            params: {
                                id: id
                            }
                        }).then(response => {
                            var data = response.data;
                            codeState(data.code, {
                                200: function () {
                                    alert("监测类别删除成功！");
                                    me.getData();
                                }
                            })
                        }, response => {
                            serverErrorInfo(response);
                        });
                    }
                })
            },
            edit(item){
                var me = this;
                router.push("/type/change?id=" + item.id);
            },
            getData(){
                var me = this;
                me.fetchData(me.currentPage, rowCount);
                me.fetchPages(rowCount);
            },
            removeAll(){
                var me = this;
                if (me.selected.length == 0) {
                    error("至少需要选择一个监测类别");
                    return;
                }
                confirm({
                    content: "是否删除当前选中监测类别？",
                    success: function () {
                        me.$http.get("/api/type/deleteAll", {
                            params: {
                                selected: me.selected
                            }
                        }).then(response => {
                            var data = response.data;
                            codeState(data.code, {
                                200: function () {
                                    alert("监测类别删除成功！");
                                    me.getData();
                                    closeConfirm();
                                }
                            });
                        }, response => {
                            serverErrorInfo(response);
                        });
                    }
                });
            },
            total(){
                var me = this;
                me.condition = "";
                me.currentPage = 1;
                me.getData();
            }
        }
    }
</script>
