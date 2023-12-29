<template>
    <v-container>
      <v-sheet elevation="1">
        <v-row>
          <v-col cols="12">
            <h2>{{ post.postTtl }}</h2>
            <p>작성자: {{ post.userId }}</p>
            <p>작성일: {{ post.rgstDt }}</p>
            <p>내용: {{ post.postCont }}</p>
            <div v-if="post.filesVO && post.filesVO.length > 0">
              <!-- 이미지가 있는 경우 -->
              <v-row>
                <v-col v-for="file in post.filesVO" :key="file.fileId" cols="3" md="3">
                  <div style="max-width: 150px; max-height: 150px; overflow: hidden;">
                    <a :href="getImageUrl(file.fileId)" download>
                        <img :src="getImageUrl(file.fileId)" :alt="file.fileOrgNm" style="max-width: 100%; height: auto;">
                    </a>
                  </div>
                    <p>{{ file.fileOrgNm }}</p>
                </v-col>
              </v-row>
            </div>
            <div v-else>
              <!-- 이미지가 없는 경우 -->
              <p>이미지가 없습니다.</p>
            </div>

            <!-- 게시글 수정 버튼 -->
            <v-spacer></v-spacer>
            <nuxt-link :to="{ path: '/board/postUpdate', query: { postId: post.postId } }">
              <v-btn color="primary">게시글 수정</v-btn>
            </nuxt-link>
            <v-spacer></v-spacer>
            <v-btn color="error" @click="deletePost">게시글 삭제</v-btn>
          </v-col>
        </v-row>
      </v-sheet>
    </v-container>
  </template>

  <script>
  export default {
    data() {
      return {
        post: {},
      };
    },
    mounted() {
      const postId = this.$route.query.postId;
      this.$axios
        .get(`/api/v1/board/postinfo/{postId}?postId=${postId}`)
        .then((resp) => {
          this.post = resp.data;
        })
        .catch((error) => {
          console.error(error);
        });
    },
    methods: {
      getImageUrl(fileId) {
          // fileId를 사용하여 이미지의 URL을 동적으로 생성
          return `/api/v1/getimage/${fileId}?fileId=${fileId}`;
      },
      deletePost() {
        const postId = this.post.postId;
        this.$axios
          .delete(`/api/v1/board/postdelete/{postId}?postId=${postId}`)
          .then(() => {
            console.log('게시글이 성공적으로 삭제되었습니다.');
            // 삭제 후 화면을 어디론가 이동하거나 다시 불러올 수 있습니다.
            this.$router.push('/board/boardList');
          })
          .catch((error) => {
            console.error('게시글 삭제 중 오류 발생', error);
          }
        );
      },
    },
  };
  </script>
