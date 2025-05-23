<template>
  <div class="feeds-container" v-infinite-scroll="loadMoreData" :infinite-scroll-distance="50">
    <Waterfall
      :list="noteList"
      :width="options.width"
      :gutter="options.gutter"
      :hasAroundGutter="options.hasAroundGutter"
      :animation-effect="options.animationEffect"
      :animation-duration="options.animationDuration"
      :animation-delay="options.animationDelay"
      :breakpoints="options.breakpoints"
      style="min-width: 740px"
    >
      <template #item="{ item }">
        <el-skeleton style="width: 240px" :loading="!item.isLoading" animated>
          <template #template>
            <el-image
              :src="item.noteCover"
              :style="{
                width: '240px',
                maxHeight: '300px',
                height: item.noteCoverHeight + 'px',
                borderRadius: '8px',
              }"
              @load="handleLoad(item)"
            >
            </el-image>

            <div style="padding: 14px">
              <el-skeleton-item variant="h3" style="width: 100%" />
              <div style="display: flex; align-items: center; margin-top: 2px; height: 16px">
                <el-skeleton style="--el-skeleton-circle-size: 20px">
                  <template #template>
                    <el-skeleton-item variant="circle" />
                  </template>
                </el-skeleton>
                <el-skeleton-item variant="text" style="margin-left: 10px" />
              </div>
            </div>
          </template>

          <template #default>
            <div class="card" style="max-width: 240px">
              <div class="image-container">
                <el-image
                  :src="item.noteCover"
                  :style="{
                    width: '240px',
                    maxHeight: '300px',
                    height: item.noteCoverHeight + 'px',
                    borderRadius: '8px',
                  }"
                  fit="cover"
                  @click="toMain(item.id)"
                >
                </el-image>
                <div v-if="item.auditStatus === '0'" class="overlay">审核中</div>
                <div v-if="item.auditStatus === '2'" class="overlay not-passed">未通过⚠️</div>
              </div>
              <div class="footer">
                <a class="title">
                  <span>{{ item.title }}</span>
                </a>
                <div class="author-wrapper">
                  <a class="author">
                    <img class="author-avatar" :src="item.avatar" />
                    <span class="name">{{ item.username }}</span>
                  </a>
                  <span class="like-wrapper like-active">
                    <i
                      class="iconfont icon-follow-fill"
                      :style="{ width: '1em', height: '1em', color: item.isLike ? 'red' : 'black' }"
                      v-if="item.isLike"
                    >
                    </i>
                    <i class="iconfont icon-follow" style="width: 1em; height: 1em" v-else></i>
                    <span class="count">{{ item.likeCount }}</span>
                  </span>
                </div>
              </div>
              <div class="top-tag-area" v-show="type === 1 && item.pinned === '1'">
                <div class="top-wrapper">置顶</div>
              </div>
            </div>
          </template>
        </el-skeleton>
      </template>
    </Waterfall>
  </div>

  <Main
    v-show="mainShow"
    :nid="nid"
    :nowTime="new Date()"
    class="animate__animated animate__zoomIn animate__delay-0.5s"
    @click-main="close"
  ></Main>
</template>

<script lang="ts" setup>
import { Waterfall } from "vue-waterfall-plugin-next";
import "vue-waterfall-plugin-next/dist/style.css";
import { ref, onMounted, watch } from "vue";
import { getTrendByUser } from "@/api/user";
import Main from "@/views/main/main.vue";
import { options } from "@/constant/constant";
import { useRoute } from "vue-router";
import { useUserStore } from "@/store/userStore";
const route = useRoute();

const props = defineProps({
  type: {
    type: Number,
    default: 1,
  },
});

watch(
  () => [props.type],
  ([newType]) => {
    currentPage.value = 1;
    noteList.value = [] as Array<any>;
    getNoteList(newType);
  }
);

const noteList = ref<Array<any>>([]);
const noteTotal = ref(0);
const uid = route.query.uid as string;
const currentPage = ref(1);
const pageSize = 10;
const nid = ref("");
const mainShow = ref(false);
const userStore = useUserStore();
const currentUid = userStore.getUserInfo().id;

const handleLoad = (item: any) => {
  item.isLoading = true;
};

const close = () => {
  mainShow.value = false;
};

const toMain = (noteId: string) => {
  // router.push({ name: "main", state: { nid: nid } });
  nid.value = noteId;
  mainShow.value = true;
};

const setData = (res: any) => {
  const { records, total } = res.data;
  noteTotal.value = total;
  // 过滤掉不是当前用户且状态“审核中”或“未通过”的记录
  const filteredRecords = records.filter((item: any) => {
    return item.uid === currentUid || (item.auditStatus !== "0" && item.auditStatus !== "2");
  });
  noteList.value.push(...filteredRecords);
};

const getNoteList = (type: number) => {
  getTrendByUser(currentPage.value, pageSize, uid, type).then((res) => {
    setData(res);
  });
};

const loadMoreData = () => {
  currentPage.value += 1;
  getNoteList(props.type);
};

const initData = () => {
  getNoteList(1);
};

onMounted(() => {
  initData();
});
</script>

<style lang="less" scoped>
.image-container {
  position: relative;
  display: inline-block;
}
.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5); /* 半透明背景 */
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  font-size: 20px;
}
.overlay.not-passed {
  color: red; /* 设置未通过状态的字体颜色为红色 */
}
.feeds-container {
  position: relative;
  transition: width 0.5s;
  margin: 0 auto;

  .noteImg {
    width: 240px;
    max-height: 300px;
    object-fit: cover;
    border-radius: 8px;
  }

  .card {
    position: relative;

    .top-tag-area {
      position: absolute;
      left: 12px;
      top: 12px;
      z-index: 4;

      .top-wrapper {
        background: #ff2442;
        border-radius: 999px;
        font-weight: 500;
        color: #fff;
        line-height: 120%;
        font-size: 12px;
        padding: 5px 8px;
        display: flex;
        align-items: center;
        justify-content: center;
      }
    }
  }

  .footer {
    padding: 12px;

    .title {
      margin-bottom: 8px;
      word-break: break-all;
      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 2;
      overflow: hidden;
      font-weight: 500;
      font-size: 14px;
      line-height: 140%;
      color: #333;
    }

    .author-wrapper {
      display: flex;
      align-items: center;
      justify-content: space-between;
      height: 20px;
      color: rgba(51, 51, 51, 0.8);
      font-size: 12px;
      transition: color 1s;

      .author {
        display: flex;
        align-items: center;
        color: inherit;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        margin-right: 12px;

        .author-avatar {
          margin-right: 6px;
          width: 20px;
          height: 20px;
          border-radius: 20px;
          border: 1px solid rgba(0, 0, 0, 0.08);
          flex-shrink: 0;
          object-fit: cover;
        }

        .name {
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
      }

      .like-wrapper {
        position: relative;
        cursor: pointer;
        display: flex;
        align-items: center;

        .count {
          margin-left: 2px;
        }
      }
    }
  }
}
</style>
