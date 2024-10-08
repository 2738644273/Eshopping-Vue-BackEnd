<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <!-- 搜索 -->
      <el-input
        v-model="query.value"
        clearable
        placeholder="输入搜索内容"
        style="width: 200px"
        class="filter-item"
        @keyup.enter.native="toQuery"
      />
      <el-select
        v-model="query.type"
        clearable
        placeholder="类型"
        class="filter-item"
        style="width: 130px"
      >
        <el-option
          v-for="item in queryTypeOptions"
          :key="item.key"
          :label="item.display_name"
          :value="item.key"
        />
      </el-select>
      <el-button
        class="filter-item"
        size="mini"
        type="success"
        icon="el-icon-search"
        @click="toQuery"
        >搜索</el-button
      >
      <!-- 新增 -->
      <el-button
        type="danger"
        class="filter-item"
        size="mini"
        icon="el-icon-refresh"
        @click="toQuery"
        >刷新</el-button
      >
    </div>
    <!--表单组件-->
    <eForm ref="form" :is-add="isAdd" />
    <!--表格渲染-->
    <el-table v-loading="loading" :data="data" size="small" style="width: 100%">
      <el-table-column prop="info.seckill.id" label="id" />
      <el-table-column prop="info.product.image" label="产品主图">
        <template slot-scope="scope">
          <a
            :href="scope.row.info.product.image"
            style="color: #42b983"
            target="_blank"
            ><img
              :src="scope.row.info.product.image"
              alt="点击打开"
              class="el-avatar"
          /></a>
        </template>
      </el-table-column>
      <el-table-column prop="info.seckill.title" label="产品标题" />
      <el-table-column prop="info.seckill.price" label="秒杀价" />
      <el-table-column prop="info.seckill.ot_price" label="原价" />
      <el-table-column prop="info.seckill.stock" label="库存" />
      <el-table-column prop="info.seckill.sales" label="销量" />
      <el-table-column prop="info.seckill.num" label="限购" />
      <el-table-column prop="info.seckill.status" label="秒杀状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.info.seckill.status"> 开启 </el-tag>
          <el-tag v-else type="info"> 关闭 </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="info.seckill.stop_time" label="开始时间段">
        <template slot-scope="scope">
          <span>
          {{JSON.parse(scope.row.time.value).time}}点-{{Number(JSON.parse(scope.row.time.value).time)+Number(JSON.parse(scope.row.time.value).continued)}}点
         </span>
        </template>
      </el-table-column>
      <el-table-column prop="info.seckill.stop_time" label="开始时间">
        <template slot-scope="scope">
          <span>{{
            parseTime(scope.row.info.seckill.start_time, "{y}-{m}-{d}")
          }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="stopTime" label="结束时间">
        <template slot-scope="scope">
          <span>{{
            parseTime(scope.row.info.seckill.stop_time, "{y}-{m}-{d}")
          }}</span>
        </template>
      </el-table-column>
      <el-table-column
        v-if="
          checkPermission([
            'admin',
            'YXSTORESECKILL_ALL',
            'YXSTORESECKILL_EDIT',
            'YXSTORESECKILL_DELETE',
          ])
        "
        label="操作"
        width="150px"
        align="center"
      >
        <template slot-scope="scope">
          <!-- <el-button v-permission="['admin','YXSTORESECKILL_ALL','YXSTORESECKILL_EDIT']" size="mini" type="primary" icon="el-icon-edit" @click="edit(scope.row)" /> -->
          <el-popover
            :ref="scope.row.info.seckill.id"
            v-permission="[
              'admin',
              'YXSTORESECKILL_ALL',
              'YXSTORESECKILL_DELETE',
            ]"
            placement="top"
            width="180"
          >
            <p>确定删除本条数据吗？</p>
            <div style="text-align: right; margin: 0">
              <el-button
                size="mini"
                type="text"
                @click="$refs[scope.row.info.seckill.id].doClose()"
                >取消</el-button
              >
              <el-button
                :loading="delLoading"
                type="primary"
                size="mini"
                @click="subDelete(scope.row.info.seckill.id)"
                >确定</el-button
              >
            </div>
            <el-button
              slot="reference"
              type="danger"
              icon="el-icon-delete"
              size="mini"
            />
          </el-popover>
        </template>
      </el-table-column>
    </el-table>
    <!--分页组件-->
    <el-pagination
      :total="total"
      :current-page="page"
      style="margin-top: 8px"
      layout="total, prev, pager, next, sizes"
      @size-change="sizeChange"
      @current-change="pageChange"
    />
  </div>
</template>

<script>
import checkPermission from "@/utils/permission";
import initData from "@/mixins/crud";
import { del } from "@/api/seckill";
import eForm from "./form";
import { formatTimeTwo, parseTime } from "@/utils/index";
export default {
  components: { eForm },
  mixins: [initData],
  data() {
    return {
      delLoading: false,
      queryTypeOptions: [{ key: "title", display_name: "产品标题" }],
    };
  },
  created() {
    this.$nextTick(() => {
      this.init();
    });
  },
  methods: {
    parseTime,
    formatTimeTwo,
    checkPermission,
    beforeInit() {
      this.url = "api/Seckill";
      const sort = "id,desc";
      this.params = { page: this.page, limit: this.size, sort: sort };
      const query = this.query;
      const type = query.type;
      const value = query.value;
      if (type && value) {
        this.params[type] = value;
      }
      return true;
    },
    subDelete(id) {
      this.delLoading = true;
      del(id)
        .then((res) => {
          this.delLoading = false;
          this.$refs[id].doClose();
          this.dleChangePage();
          this.init();
          this.$notify({
            title: "删除成功",
            type: "success",
            duration: 2500,
          });
        })
        .catch((err) => {
          this.delLoading = false;
          this.$refs[id].doClose();
          this.init();
        });
    },
    add() {
      this.isAdd = true;
      this.$refs.form.dialog = true;
    },
    edit(data) {
      this.isAdd = false;
      const _this = this.$refs.form;

_this.form = {
        id: data.id,
        productId: data.productId,
        image: data.image,
        images: data.images,
        imageArr: data.image.split(","),
        sliderImageArr: data.images.split(","),
        title: data.title,
        info: data.info,
        price: data.price,
        cost: data.cost,
        otPrice: data.otPrice,
        giveIntegral: data.giveIntegral,
        sort: data.sort,
        stock: data.stock,
        sales: data.sales,
        unitName: data.unitName,
        postage: data.postage,
        description: data.description,
        startTimeDate: new Date(data.startTimeDate),
        endTimeDate: new Date(data.endTimeDate),
        addTime: data.addTime,
        status: data.status,
        isPostage: data.isPostage,
        isHot: data.isHot,
        isDel: data.isDel,
        num: data.num,
        isShow: data.isShow,
        timeId: data.timeId,
      };
      _this.dialog = true;
    },
  },
};
</script>

<style scoped>
</style>
