<template>
    <section style="padding: 21px 16px;margin: 0 10px;background: #fff">
        <sticky class-name="sub-navbar" :fixedDom="fixedDom" stickyTop='50' zIndex='90' v-on:topShow="topShowFn" v-if="suctionTopVisible">
            <ul style="display: flex;width:100%" v-show="topShow">
                <template v-for="items in filterTableItems" >
                    <li
                        :style="tableitem.width ? {width:tableitem.width+'px',textAlign:'center',lineHeight:'44px',color:'#909399',fontWeight:'500',backgroundColor: '#F4F8FF'} : isWidthStyle"
                        v-for="(tableitem,index) in items.subs"
                    >{{tableitem.label}}</li>
                </template>
            </ul>
        </sticky>
        <!--列表-->
        <el-table
                :key="key"
                :stripe="stripe"
                :loading="loading"
                :data="tableData"
                border
                resizable="false"
                highlight-current-row
                style="width:100%;"
                :max-height="tableMaxHeight"
                v-loading="loading"
                @sort-change="sortChange"
                @row-click = "rowClickFn"
                :row-style="rowStyle"
                ref="refTable">
            <template v-for="(items,index) in filterTableItems">
                <el-table-column
                        v-for="(tableitem,index) in items.subs"
                        :type="tableitem.columnType"
                        :label="tableitem.label"
                        :header-align="tableitem.headerAlign"
                        :align="tableitem.align || 'center'"
                        :sortable="false"
                        :width="tableitem.width"
                        :formatter="tableitem.format"
                        :fixed="tableitem.fixed"
                >
                    <template slot-scope="scope">
                       <table-expand
                               :format-collectors="formatCollectors"
                               :pname="pname"
                               :single-double-type="singleDoubleType"
                               :expand-data="scope.row"
                               :expand-label="TableItems"
                               :name-type="tableitem.nameType"
                               v-if="tableitem.columnType == 'expand'"></table-expand>
<!--                        自定义拓展-->
                        <expand
                                v-else-if="tableitem.columnType === 'render'"
                                :row="scope.row"
                                :column="tableitem"
                                :index="scope.$index"
                                :render="tableitem.render"></expand>
<!--                        列表树-->
                        <span v-else-if="tableitem.hasChildren && scope.row.children">
                                <span v-if="scope.row.children.length>0">
                                    <span
                                            v-for="(space, levelIndex) in scope.row._level"
                                            class="ms-tree-space"></span>
                                    <i class="el-icon-caret-right" v-if="scope.row._expanded" @click="toggle(scope.$index)"></i>
                                    <i class="el-icon-caret-bottom" v-else="scope.row._expanded" @click="toggle(scope.$index)"></i>
                                    {{scope.row[tableitem.prop]}}
                                </span>
                                <span v-else>
                                    <span
                                            v-for="(space, levelIndex) in scope.row._level"
                                            class="ms-tree-space"></span>{{scope.row[tableitem.prop]}}
                                </span>
                        </span>

                        <span v-else> {{scope.row[tableitem.prop]}}</span>
                    </template>
                </el-table-column>
            </template>


        </el-table>
        <div class="pagination-pos">
            <!--工具条-->
            <el-col :span="24"  align="bottom" style="margin-top:5px;margin-bottom:5px">
                <el-col :span="24" align="right">
                    <el-pagination
                            @size-change="handleSizeChange"
                            @current-change="handleCurrentChange"
                            :current-page="currentPage"
                            :page-sizes="[20, 40, 80]"
                            :page-size="pageSize"
                            layout="total, sizes, prev, pager, next, jumper"
                            background
                            :total="total"></el-pagination>
                </el-col>
            </el-col>
        </div>
        <!--删除提示框-->
        <el-dialog
                width="478px"
                :visible.sync="delForm.delVisible"
                :show-close="false"
                custom-class="custom-dialog deleteTip">

            <header class="dialog-header" slot="title">
                提示<i class="el-icon-close dialog-header-iconfont" @click="cancelDel"></i>
            </header>
            <div class="dialog-body" style="height: 40px;line-height: 40px;text-align: center;font-size: 16px">
                 <p><img class="info-icon" :src="infoIcon"> 确认删除？此操作不可恢复！</p>
            </div>
            <footer slot="footer" class="dialog-footer">
                <el-button type="primary" class="dialog-footer-btn" @click="handledelete" >确 定</el-button>
                <el-button @click="cancelDel" class="dialog-footer-btn" style="margin-left: 36px">取 消</el-button>
            </footer>
        </el-dialog>
        <!--没封装数据的删除提示框-->
        <el-dialog
                width="478px"
                :visible.sync="delForm.delVisible2"
                :show-close="false"
                custom-class="custom-dialog deleteTip">

            <header class="dialog-header" slot="title">
                提示<i class="el-icon-close dialog-header-iconfont" @click="cancelDel"></i>
            </header>
            <div class="dialog-body" style="height: 40px;line-height: 40px;text-align: center;font-size: 16px">
                <p><img class="info-icon" :src="infoIcon"> 确认删除？此操作不可恢复！</p>
            </div>
            <footer slot="footer" class="dialog-footer">
                <el-button type="primary" class="dialog-footer-btn" @click="handledelete2" >确 定</el-button>
                <el-button @click="cancelDel" class="dialog-footer-btn" style="margin-left: 36px">取 消</el-button>
            </footer>
        </el-dialog>
        <!--添加-->
        <custom-add-form
                v-if="addTo"
                ref="addref"
                :dialogStyle="dialogStyle"
                :value="addRowData"
                :addFormConfig="TableItems"
                :title="addTitle"
                :loading="addLoading"
                v-on:input="onAddInput"
                v-on:add="onAdd"
                v-on:cancelAdd="cancelAdd"
                :addVisible="addFormVisible"></custom-add-form>
        <!--编辑-->
        <custom-edit-form
                v-if="editTo"
                ref="editref"
                :dialogStyle="dialogStyle"
                :value="editRowData"
                :editFormConfig="TableItems"
                title="编辑"
                :loading="editLoading"
                v-on:input="onEditInput"
                v-on:edit="onEdit"
                v-on:cancelEdit="cancelEdit"
                :editVisible="editFormVisible"></custom-edit-form>
    </section>
</template>

<script>
    import {mapState} from 'vuex';
    import {path,bolinkPath} from '../../api/api';
    import { getTableQuery,delTableData ,editTableData,addTableData} from '../../api/base'
    import common from '../../common/js/common';
    import Printd from 'printd';
    import axios from 'axios'
    import TableExpand from './TableExpand'
    import customEditForm from '../edit-form/editForm'
    import customAddForm from '../add-form/addForm'
    import expand from './expand'
    import ElButton from 'element-ui/packages/button/src/button';
    import Sticky from './sticky'
    import dataTranslate from '../../common/js/dataTranslate'
    import qs from 'Qs'
    export default {
        components: {
            ElButton,
            TableExpand,
            expand,
            Sticky,
            customEditForm,
            customAddForm
        },
        data(){
            return {
                key:0,
                // tableMaxHeight:'auto',
                editLoading:false,
                addLoading:false,
                rowdata:{},
                editFormVisible:false,
                addFormVisible:false,
                addFormData:{},
                off:true,
                loading:false,
                topShow:false,
                zIndex:999,
                stickyTop:50,
                width:undefined,
                height:undefined,
                active:false,
                position:'fixed',
                isSticky:false,
                wHeight:undefined,
                isWidthStyle:"flex:1;text-align:center;line-height:44px;color:#909399;font-weight:500;background-color: #F4F8FF;",
                currentHeight:'300px',
                orderby:'desc',
                pageSize:20,
                currentPage:1,
                total:0,
                tableData:[],
                sform:{}, //搜索的条件
                infoIcon:require('../../assets/images/info-icon.png'),
            }
        },
        props:{
            isTree:{
                type:Boolean,
                default:false,
            },
            defaultExpandAll:{
              type:Boolean,
              default:true,
            },
            isBolink:{
              type:Boolean,
              default:false,
            },
            openFixedHeight:{
                type:Boolean,
                default:false,
            },
            stripe:Boolean,
            TableItems:{
                type:Array,
                default:[],
            },
            editRowData:Object,
            addRowData:Object,
            formatCollectors:Array,
            pname:Array,
            singleDoubleType:Array,
            alignPos:String,
            btsWidth:String,
            queryapi:String,
            editapi:String,
            addapi:String,
            addTo:Number,
            editForm:Object,
            editTo:Number,
            exportapi:String,
            delapi:String,
            fieldsstr:String,
            orderfield:{
                type:String,
                default:'id'
            },
            fixedDom:String,
            searchForm:Object,
            delForm:{
                type:Object,
                default:()=>{
                    return {
                        delVisible:false,
                        delVisible2:false
                    }

                }
            },
            dialogStyle:Object,
            addTitle:String,
            addedValue:{
                type:Object,
                default:()=>{
                    return {

                    }

                }
            },
            suctionTopVisible:{
                type:Boolean,
                default:true,
            }
        },
        methods:{
            rowStyle(rows, rowIndex) {
                let row = rows.row;
                let show = (row._parent ? ((!row._parent._expanded) && row._parent._show) : true)
                row._show = show
                return show ? '' : 'display:none;'
            },
            toggle: function (trIndex) {
                let that = this;
                let record = that.tableData[trIndex];
                record._expanded = !record._expanded;
            },
            //编辑
            cancelEdit(){
                this.editFormVisible = false;
            },
            onEdit: function () {
                //发送ajax,提交表单更新
                /**
                 * 通过isBolink 的值，选用不同的访问路径
                 * isBolink  true为泊链的访问地址   false为云平台的访问地址
                 *path--->云平台
                 * bolinkPath --- 泊链
                 */

                if(this.isBolink){
                    let that = this;
                    let api = this.editapi;
                    let eform = this.rowdata;
                    that.editLoading = true;
                    this.$refs.editref.$refs.editForm.validate((valid) => {
                        if (valid) {
                            let formData  = new FormData();
                            formData.append('token',sessionStorage.getItem('token'));
                            if(eform){
                                for(let item in eform){
                                    formData.append(item,eform[item])
                                }
                            }
                            axios.post(bolinkPath+api,formData).then(res=>{
                                that.$message({
                                    message: '编辑成功!',
                                    type: 'success',
                                    duration: 600
                                });
                                setTimeout(()=>{
                                    that.editFormVisible = false;
                                    that.getTableData(that.sform,that);
                                },60)

                            }).catch(err => {
                                that.$message({
                                    message: '更新失败',
                                    type: 'error',
                                    duration: 600
                                });
                                that.editLoading = false;
                            })
                        }else{
                            that.editLoading = false;
                        }
                    });
                }else{
                    let that = this;
                    let api = this.editapi;
                    let eform = this.rowdata;
                    eform = common.generateForm(eform);
                    that.editLoading = true;
                    this.$refs.editref.$refs.editForm.validate((valid) => {
                        if (valid) {
                            editTableData(api,eform).then(res=>{
                                if(res.status == 200){
                                    if(res.data.state == 1){
                                        that.$message({
                                            message: '编辑成功!',
                                            type: 'success',
                                            duration: 600
                                        });
                                        setTimeout(()=>{
                                            that.editFormVisible = false;
                                            that.getTableData(that.sform,that);
                                        },60)
                                    }else{
                                        that.$message({
                                            message: res.data.msg,
                                            type: 'info',
                                            duration: 600
                                        });
                                        that.editLoading = false;
                                    }
                                }
                            }).catch(err => {
                                that.$message({
                                    message: '更新失败',
                                    type: 'error',
                                    duration: 600
                                });
                                that.editLoading = false;
                            })
                        }else{
                            that.editLoading = false;
                        }
                    });
                }



            },
            onEditInput:function (eform) {
                this.rowdata=eform;
                this.$emit('editInput',this.rowdata)
            },
            //添加
            cancelAdd:function () {
                this.addFormVisible = false;
            },
            onAdd:function () {
                //发送请求,添加一条记录
                /**
                 * 通过isBolink 的值，选用不同的访问路径
                 * isBolink  true为泊链的访问地址   false为云平台的访问地址
                 *path--->云平台
                 * bolinkPath --- 泊链
                 */

                let that = this;
                let api = this.addapi;
                that.addLoading = true;
                this.$refs.addref.$refs.addForm.validate((valid) => {
                    if (valid) {
                        that.addLoading = true;
                        if(that.isBolink){
                            let aform = this.addFormData;
                            let formData  = new FormData();
                            formData.append('token',sessionStorage.getItem('token'));
                            if(aform){
                                for(let item in aform){
                                    formData.append(item,aform[item])
                                }
                            }
                            axios.post(bolinkPath+api,formData).then(res=>{
                                that.$message({
                                    message: '添加成功!',
                                    type: 'success',
                                    duration: 600
                                });
                                setTimeout(()=>{
                                    that.addFormVisible = false;
                                    that.getTableData(that.addedValue,that);
                                    that.$emit('clearSearchData');
                                },60);

                            }).catch(error => {
                                that.$message({
                                    message: '更新失败',
                                    type: 'error',
                                    duration: 600
                                });
                                that.addLoading = false;
                            })
                        }else{
                            let aform = this.addFormData;
                            aform = common.generateForm(aform);
                            addTableData(api,aform).then(res=>{
                                if(res.status == 200){
                                    if(res.data.state == 1){
                                        that.$message({
                                            message: '添加成功!',
                                            type: 'success',
                                            duration: 600
                                        });
                                        setTimeout(()=>{
                                            that.addFormVisible = false;
                                            that.getTableData(that.addedValue,that);
                                            that.$emit('clearSearchData');
                                        },60);
                                    }else{
                                        that.addLoading = false;
                                        that.$message({
                                            message: res.data.msg,
                                            type: 'info',
                                            duration: 600
                                        });
                                    }
                                }
                            }).catch(err => {
                                that.$message({
                                    message: '更新失败',
                                    type: 'error',
                                    duration: 600
                                });
                                that.addLoading = false;
                            })
                        }

                    }else{
                        that.addLoading = false;
                    }
                });
            },
            onAddInput(aform) {
                this.addFormData = aform;
                this.$emit('addInput',this.addFormData)
            },
            topShowFn(val){
                this.topShow = val;
            },
            sortChange(){

            },
            handleSizeChange(val){
                this.pageSize = val;
                this.currentPage = 1;
                this.getTableData(this.sform,this);
            },
            handleCurrentChange(val){
                this.currentPage = val;
                this.getTableData(this.sform,this);
            },
            rowClickFn(row, event, column){
                // this.$refs['refTable'].toggleRowExpansion(row)
            },
            getTableData(sform,that){
                var url = that.queryapi;
                that.loading = true;
                this.tableData=[];

                /**
                 * 通过isBolink 的值，选用不同的访问路径
                 * isBolink  true为泊链的访问地址   false为云平台的访问地址
                 *path--->云平台
                 * bolinkPath --- 泊链
                 */

                if(that.isBolink){
                    let formData  = new FormData();
                    formData.append('rp',this.pageSize);
                    formData.append('page',this.currentPage);
                    formData.append('orderby',this.orderby);
                    formData.append('fieldsstr',this.fieldsstr);
                    formData.append('orderfield',this.orderfield);
                    formData.append('token',sessionStorage.getItem('token'));
                    if(sform){
                        for(let item in sform){
                            formData.append(item,sform[item])
                        }
                    }
                    axios.post(bolinkPath+url,formData).then(response=>{
                        that.loading = false;
                        that.off = true;
                        if(response.status == 200){
                            let tableData =  response.data;
                            this.total = tableData.total;
                            if(tableData.validate == 1){
                                this.$alert('登录超时，请重新登录', '登录超时提示', {
                                    confirmButtonText: '确定',
                                    callback: action => {
                                        sessionStorage.removeItem('user');
                                        sessionStorage.removeItem('token');
                                        localStorage.removeItem('comid');
                                        localStorage.removeItem('groupid');
                                        that.$router.push('/login');
                                    }
                                });
                            }else if(tableData.rows[0] == null){
                                this.tableData = [];
                            }else{
                                this.tableData= tableData.rows;
                                if(this.isTree){
                                    this.tableData = dataTranslate.treeToArray(this.tableData, null, null, this.defaultExpandAll)
                                }

                            }
                            this.$emit('transferData',tableData);
                        }else{
                            this.total = 0;
                            this.tableData= [];
                            this.$emit('transferData',[]);
                        }
                    }).catch(error => {
                        that.loading = false;
                        that.off = true;
                        console.log('error--->',error)
                    })
                }else{
                    var nform = sform;
                    nform.rp = this.pageSize;
                    nform.page = this.currentPage;
                    nform.orderby = this.orderby;
                    nform.fieldsstr = this.fieldsstr;
                    nform.orderfield = this.orderfield;
                    nform = common.generateForm(nform);
                    getTableQuery(url,nform).then(response =>{
                        that.loading = false;
                        that.off = true;
                        if(response.status == 200){
                            let tableData =  response.data
                            this.total = tableData.total;
                            this.tableData= tableData.rows;
                            this.$emit('transferData',tableData);
                        }
                    }).catch(err => {
                        that.loading = false;
                        that.off = true;
                        console.log('拉取错误了',err)
                    })
                }
            },
            //导出表格数据
            handleExport() {
                let vm = this;
                let api = this.exportapi;
                let params = '';
                if (common.getLength(this.sform) == 0) {
                    params = 'fieldsstr=' + this.fieldsstr + '&token=' + sessionStorage.getItem('token');
                } else {
                    for (var x in this.sform) {
                        if(x=='car_number'||x=='nickname1'){
                            params += x + '=' + encodeURI(encodeURI(this.sform[x])) + '&';
                        }else{
                            params += x + '=' + this.sform[x] + '&';
                        }

                    }
                    params += 'token=' + sessionStorage.getItem('token')+'&fieldsstr='+this.fieldsstr;
                }
                let groupid = sessionStorage.getItem('groupid');
                let cityid = sessionStorage.getItem('cityid');
                if (groupid != 'undefined' && !(params.indexOf('groupid=') > -1)) {
                    params += '&groupid=' + groupid;
                }
                if (cityid != 'undefined' && !(params.indexOf('cityid=') > -1)) {
                    params += '&cityid=' + cityid;
                }
                if(this.isBolink){
                    window.open(bolinkPath + api + '?' + params);
                }else{
                    if (params.indexOf('comid=') > -1) {
                        window.open(path + api + '?' + params);
                    } else {
                        window.open(path + api + '?' + params + '&comid=' + sessionStorage.getItem('comid'));
                    }
                }


            },
            //取消删除
            cancelDel(){
                this.$emit('cancelDel',false)
            },
            //删除
            handledelete() {
                let that = this;
                let url = this.delapi;
                if(that.isBolink){
                    this.$axios.post(bolinkPath+url+'?id='+this.delForm.id+'&token='+sessionStorage.getItem('token'))
                        .then(res=>{
                            let ret = res.data;
                            if(ret>0){
                                that.$emit('cancelDel',false);
                                that.tableData.splice(that.delForm.$index,1);
                                that.total = that.total - 1;
                                that.$message({
                                    message:'删除成功',
                                    type: 'success'
                                });
                                that.$emit('totalCount',that.total);
                            }else{
                                that.$message({
                                    message: '更新失败',
                                    type: 'warning'
                                });
                            }
                        })
                        .catch(err=>{
                            that.$message({
                                message: '网络错误，请稍后重试',
                                type: 'warning'
                            });
                        })
                }else{
                    let dform = JSON.parse(JSON.stringify( this.delForm ));
                    dform.token = sessionStorage.getItem('token');
                    dform = common.generateForm(dform);
                    delTableData(url,dform).then(response=>{
                        if(response.data.state == 1){
                            that.$emit('cancelDel',false);
                            that.tableData.splice(that.delForm.$index,1);
                            that.total = that.total - 1;
                            let msg = '';
                            if(response.data.msg == undefined){
                                msg = '删除成功!';
                            }else {
                                msg = response.data.msg;
                            }
                            that.$message({
                                message:msg,
                                type: 'success'
                            });
                            that.$emit('totalCount',that.total);
                        }else{
                            that.$message({
                                message: response.data.msg,
                                type: 'warning'
                            });
                        }
                    }).catch(err=>{
                        that.$message({
                            message: '网络错误，请稍后重试',
                            type: 'warning'
                        });
                    })
                }

            },

            //没封装的删除
            handledelete2() {
                let that = this;
                let url = this.delapi;
                let dform = JSON.parse(JSON.stringify( this.delForm ));
                dform.token = sessionStorage.getItem('token');
                dform = common.generateForm(dform);
                delTableData(url,dform).then(response=>{
                    if(response.data == 1){
                        that.$emit('cancelDel',false)
                        that.tableData.splice(that.delForm.$index,1);
                        that.total = that.total - 1;
                        that.$message({
                            message:'已删除' ,
                            type: 'success'
                        });
                    }else{
                        that.$message({
                            message: '删除失败',
                            type: 'warning'
                        });
                    }

                }).catch(err=>{
                    that.$message({
                        message: '网络错误，请稍后重试',
                        type: 'warning'
                    });
                })
            },
        },
        updated(){

        },
        computed:{
            ...mapState({
                tableMaxHeight:state => state.tableMaxHeight
            }),
            tableMaxHeight:{
                get(){
                    if(this.openFixedHeight){
                        return this.$store.state.app.tableMaxHeight;
                    }else{
                        return 'auto';
                    }

                },
                set(){
                    if(this.openFixedHeight){
                        return this.$store.state.app.tableMaxHeight;
                    }else{
                        return 'auto';
                    }
                }
                // return this.$store.state.app.tableMaxHeight;
            },
            filterTableItems:function () {
                return this.TableItems.filter(function(item){
                    return (item.subs[0].hidden != true) && (item.subs[0].hidden != 'true')
                })
            }
        },
        activated(){

        },
        mounted(){
            if(this.openFixedHeight){
                this.tableMaxHeight = this.$store.state.app.tableMaxHeight;
            }else{
                this.tableMaxHeight = 'auto'
            }
            this.key = 0;
        },
        beforeMount(){
            // this.currentHeight = common.gwh() - 100 +'px';
        },
        watch:{
            tableMaxHeight:function(val){
                this.key = this.key++;
            },
            searchForm:{
                handler(newVal,oldVal){
                    if(this.off){
                        this.sform = newVal;
                        this.currentPage = 1;
                        this.off = false;
                        this.getTableData(this.sform,this)
                    }else{

                    }
                },
                deep:true
            },
            addTo:function (newVal,oldVal) {
                if(newVal != 0){
                    this.addFormVisible = true;
                }
            },
            editTo:function (newVal,oldVal) {
                if(newVal != 0){
                    this.rowdata = this.editRowData;
                    this.editFormVisible = true;
                }
            }
        }
    };

</script>

<style>
    ul{
        list-style: none;
        margin: 0;
        padding: 0;
    }
    ul li{
        margin: 0;
        padding: 0;
    }
    .pagination-pos{
        width: 100%;
        margin-top: 10px;
        margin-bottom: 20px;
        background: #fff;
        padding-bottom: 10px;
    }
    .ms-tree-space {
        position: relative;
        top: 1px;
        display: inline-block;
        font-family: 'Glyphicons Halflings';
        font-style: normal;
        font-weight: 400;
        line-height: 1;
        width: 18px;
        height: 14px;
    }

    .ms-tree-space::before {
        content: ""
    }
</style>
