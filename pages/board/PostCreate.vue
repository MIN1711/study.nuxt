<!-- PostCreate.vue -->
<template>
  <v-container>
    <v-sheet elevation="1">
      <!-- 게시글 등록을 위한 입력 폼 -->
      <v-row>
        <v-col cols="12">
          <v-form @submit.prevent="createPost">
            <v-row>
              <v-col cols="6">
                <!-- Title field -->
                <v-text-field v-model="post.postTtl" name="postTtl" label="게시글 제목"></v-text-field>
              </v-col>
              <v-col cols="6">
                <!-- Author field -->
                <v-text-field v-model="post.userId" name="userId" label="작성자"></v-text-field>
              </v-col>
            </v-row>
            <v-textarea v-model="post.postCont" name="postCont" label="게시글 내용"></v-textarea>

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

          <v-btn class="mr-auto" type="submit" color="primary">게시글 등록</v-btn>
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
      postTtl: '',     // 게시글 제목
      postCont: '',    // 게시글 내용
      userId: '',      // 작성자 ID
      files: [],       // 첨부 파일 목록
      files2: [],       // 첨부 파일 목록
    },
  };
},
methods: {
  async createPost() {
    try {

      // FormData 객체 생성
      const formData = new FormData();

      // 텍스트 데이터 추가
      formData.append('postTtl', this.post.postTtl);
      formData.append('postCont', this.post.postCont);
      formData.append('userId', this.post.userId);

      // 파일 데이터 추가
      let name = null;

      if (this.post.files.length > 0){
        for (let i = 0; i < this.post.files.length; i++) {
          // if (this.post.files2[i][0]) {
          if (this.post.files2 && this.post.files2[i] && this.post.files2[i][0]) {
            name = this.post.files2[i][0].name
            formData.append('files', this.post.files2[i][0], name);
          } else {
            formData.append('files', '');
          }
          /*
          blob = new Blob([JSON.stringify(this.post.files2[i])], {
          type: 'multipart/form-data',
          });
          */
        }
      } else {
        formData.append('files', '');
      }


      /*
      for (let i = 0; i < this.post.files.length; i++) {
        console.log(this.post.files2[i]);
        formData.append('files', this.post.files2[i]);
      }
      */

      // console.log(formData.get('files'));

      // const formData = new FormData(document.querySelector('#form'));

      // Axios를 사용하여 서버로 POST 요청 전송
      /*
      await this.$axios.post('/api/v1/board/posting', formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        },
      });
      */
      await this.$axios.post('/api/v1/board/posting', formData);

      console.log("게시글이 성공적으로 등록되었습니다");

      // 게시글 등록 후 화면을 '/boardList'로 이동
      this.$router.push('/board/boardList');
    } catch (error) {
        console.error("게시글 등록 중 오류 발생", error);
        // 오류 발생 시 필요한 작업 수행
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
  },
};
</script>
