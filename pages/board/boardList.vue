<template>
  <v-container>
      <v-sheet elevation="1">
          <v-row>
              <v-col cols="12" md="5">
                  <v-text-field v-model="srchData.search" :loading="loading" label="검색어" density="compact" variant="outlined"></v-text-field>
              </v-col>
              <v-col cols="12" md="5">
                <div>
                  <select v-model="srchData.ctgr" :loading="loading" label="카테고리" density="compact" variant="outlined">
                    <option value="">선택</option>
                    <option value="title">제목</option>
                    <option value="cont">내용</option>
                    <option value="user">작성자</option>
                  </select>
                </div>
              </v-col>
              <v-col cols="12" md="2">
                  <v-btn class="mt-2 mr-2" @click="search">검색</v-btn>
                  <v-btn class="mt-2" color="primary" @click="write">등록</v-btn>
              </v-col>
          </v-row>
      </v-sheet>
      <v-sheet>
              <v-table>
                  <thead>
                      <tr>
                          <th>#</th>
                          <th>제목</th>
                          <th>작성자</th>
                          <th>작성일</th>
                      </tr>
                  </thead>
                  <tbody>
                      <template v-if="boards != null && boards.length > 0">
                          <tr v-for="(board, idx) in boards" :key="board.postId">
                              <td>{{ idx+1 }}</td>
                              <td>
                                  <!-- <router-link :to="{path: '/postDetail', query:{postId : board.postId} }">{{ board.postTtl }}</router-link> -->
                                  <nuxt-link :to="{path: '/board/postDetail', query:{postId : board.postId} }">{{ board.postTtl }}</nuxt-link>
                                </td>
                              <td>{{ board.userId }}</td>
                              <td>{{ board.rgstDt }}</td>
                          </tr>
                      </template>
                      <template v-else>
                          <tr>
                              <td colspan="5" class="text-center">등록된 게시글이 없습니다.</td>
                          </tr>
                      </template>
                  </tbody>
              </v-table>
      </v-sheet>
  </v-container>
</template>

<script>
  export default {
      data() {
          return {
              boards: [],
              srchData : {
                  search: '',
                  ctgr: '',
              }
          }
      },
      mounted() {
          this.$axios.get('/api/v1/board/board')
          .then((resp) => {
              this.boards = resp.data;
          }).catch(() => {
              console.error(arguments);
          })
      },

      methods: {
          async search() {
              this.loading = true
              try{
                  const resp = await this.$axios.get('/api/v1/board/board', {
                      params : this.srchData
                  });
                  this.boards = resp.data;
              }catch(e) {
                  console.log(e);
              }
          },
          write(){
              this.$router.push("/board/postCreate");
          }
      }
  }
</script>
