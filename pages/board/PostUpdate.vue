<template>
  <v-container>
    <v-sheet elevation="1">
      <!-- 게시글 수정을 위한 입력 폼 -->
      <v-row>
        <v-col cols="12">
          <v-form @submit.prevent="updatePost">
            <v-row>
              <v-col cols="6">
                <!-- 제목 필드 -->
                <v-text-field v-model="post.postTtl" label="게시글 제목"></v-text-field>
              </v-col>
              <v-col cols="6">
                <!-- 작성자 필드 -->
                <v-text-field v-model="post.userId" label="작성자"></v-text-field>
              </v-col>
            </v-row>
            <v-textarea v-model="post.postCont" label="게시글 내용"></v-textarea>
            <div v-if="post.filesVO && post.filesVO.length > 0">
              <!-- 이미지가 있는 경우 -->
              <v-row>
                <v-col v-for="file in post.filesVO" :key="file.fileId" cols="3" md="3">
                  <div style="max-width: 150px; max-height: 150px; overflow: hidden;">
                    <img :src="getImageUrl(file.fileId)" :alt="file.fileOrgNm" style="width: 100%; height: auto;">
                  </div>
                  <p>{{ file.fileOrgNm }}</p>
                  <v-btn color="error" @click="delImg(file.fileId)">이미지 삭제</v-btn>
                </v-col>
              </v-row>
            </div>
            <div v-else>
              <!-- 이미지가 없는 경우 -->
              <p>이미지가 없습니다.</p>
            </div>

            <!-- 이미지를 첨부하는 입력란 -->
            <v-row v-for="(file, index) in post.files" :key="index">
              <v-col cols="10">
                <v-file-input v-model="post.files2[index]" name="files" label="이미지 첨부" multiple></v-file-input>
              </v-col>
              <v-col cols="2">
                <v-btn icon @click="removeImage(index)">
                <v-icon>mdi-delete</v-icon>
                </v-btn>
              </v-col>
            </v-row>

            <v-btn color="primary" @click="addImage">이미지 추가</v-btn>

            <v-btn class="ml-auto" type="submit" color="primary">게시글 수정</v-btn>
          </v-form>
        </v-col>
      </v-row>
    </v-sheet>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      post: {
        postId: this.$route.query.postId,
        postTtl: '',     // 게시글 제목
        postCont: '',    // 게시글 내용
        userId: '',      // 작성자 ID
        files: [],       // 첨부 파일 목록
        files2: [],      // 첨부 파일 목록
        filesVO: '',
        deletedFileIds: '',
      },
    };
  },
  mounted() {
    // 라우트 쿼리에서 postId를 사용하여 게시글 상세 정보 가져오기
    const postId = this.$route.query.postId;
    this.$axios
      .get(`/api/v1/board/postupdate/{postId}?postId=${postId}`)
      .then((resp) => {
        // 폼 필드에 가져온 게시글 상세 정보 채우기
        this.post.postTtl = resp.data.postTtl;
        this.post.postCont = resp.data.postCont;
        this.post.userId = resp.data.userId;
        this.post.filesVO = resp.data.filesVO; // 이 부분 수정
      })
      .catch((error) => {
        console.error(error);
      });
  },
  methods: {
    async updatePost() {
      try {
        const formData = new FormData();
        // 게시글 아이디 추가
        formData.append('postId', this.post.postId);

        // 삭제파일 아이디 추가
        formData.append('deletedFileIds', this.post.deletedFileIds);

        // 텍스트 데이터 추가
        formData.append('postTtl', this.post.postTtl);
        formData.append('postCont', this.post.postCont);
        formData.append('userId', this.post.userId);

        // 파일 데이터 추가
        let name = null;

        if (this.post.files.length > 0){
          for (let i = 0; i < this.post.files.length; i++) {
            if (this.post.files2 && this.post.files2[i] && this.post.files2[i][0]) {
              name = this.post.files2[i][0].name
              formData.append('files', this.post.files2[i][0], name);
            } else {
              formData.append('files', '');
            }
          }
        } else {
          formData.append('files', '');
        }
        console.log(formData);

        await this.$axios.post('/api/v1/board/postupdate.do', formData);

        console.log("게시글이 성공적으로 등록되었습니다");

        // 게시글 수정 후 화면을 '/boardDetail'로 이동
        this.$router.push('/board/boardList');

      } catch (error) {
        console.error("게시글 수정 중 오류 발생", error);
        // 필요한 경우 오류 처리
      }
    },
    addImage() {
      // 새로운 파일 입력 필드 추가
      this.post.files.push('');
    },
    removeImage(index) {
      // 지정된 파일 입력 필드 제거
      this.post.files.splice(index, 1);
    },
    getImageUrl(fileId) {
      // fileId를 사용하여 이미지의 URL을 동적으로 생성
      return `/api/v1/getimage/${fileId}?fileId=${fileId}`;
    },
    delImg(fileId) {
      // 이미지 삭제 버튼 클릭 시 호출되는 메소드
      // post.filesVO에서 해당 이미지 제거
      this.post.filesVO = this.post.filesVO.filter(file => file.fileId !== fileId);
      // deletedFileIds에 fileId 추가 (이미 추가된 경우 중복 방지)
      if (!this.post.deletedFileIds.includes(fileId)) {
        if (this.post.deletedFileIds) {
          this.post.deletedFileIds += `,${fileId}`;
        } else {
          this.post.deletedFileIds = fileId.toString();
        }
      }
    },
  },
};
</script>
