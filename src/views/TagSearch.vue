<template>
  <v-container>
    <span class="headline">{{ header }} 검색결과 확인</span>
    <p class="headline mt-3">검색 옵션</p>
    <v-btn-toggle v-model="text" flat>
      <v-btn text value="title" @click="setTitle">제목 검색</v-btn>
      <v-btn text value="content" @click="setContent">내용 검색</v-btn>
      <v-btn text value="tag" @click="setTag">태그 검색</v-btn>
    </v-btn-toggle>
    <div class="mt-3 headline">
    {{ msg }}
    
    </div>
    <v-flex v-for="i in posts.length" v-bind:key='i' class="xs" px-3>
			<div class="post-con">

			<!-- 포스트 내용 -->
			<div class="post-con-left">
					<div class="post-time py-2">
            <v-layout justify-space-between>
                <span class="grey--text">{{posts[i-1].postdate}}</span>
            </v-layout>
					</div>
				<Post
					:email="posts[i-1].email"
					:id="posts[i - 1].id"
					:date="posts[i - 1].created_at"
					:title="posts[i - 1].title"
					:content="posts[i - 1].content"
          :subject="posts[i - 1].subject"
					></Post>
			  </div>
			</div>
      <!-- <v-divider></v-divider> -->
    </v-flex>
  </v-layout>
  </v-container>
</template>

<script>
import FirebaseService from "../services/FirebaseService";
import { firestore } from "firebase";
import Post from "../components/SearchPost";
import userInfo from "../components/userInfo";

export default {
  components: {
    Post,
    userInfo
  },
  data() {
    return {
      text: "",
      msg: "",
      option: "",
      header: "",
      subjects: ["AI", "Bigdata", "Blockchain", "Webmobile"],
      posts: [],
      beforefilter: []
    };
  },
  created() {
    // console.log(this.$route,'ㅎㅁㄴㅇㅎ')
    this.header = this.$route.params.item;
    this.option = this.$route.query.options;

    if (this.option === "tag") {
      this.text = "tag";
      this.msg = "해당 태그가 등록된 게시글을 검색합니다";
      this.setTag()
    } else if (this.option === "title") {
      this.text = "title";
      this.msg = "해당 태그가 등록된 게시글을 보여드립니다.";
      this.setTitle()
    }
  },
  methods: {
    async setTag() {
      this.posts = [];
      this.msg = "해당 태그가 등록된 게시글을 보여드립니다.";

      // 먼저 태그의 post_token을 불러온다.
      const res = await FirebaseService.getTag(this.header);
      console.log(res, "zsdfszdf");
      // 토큰이 리스트에 담겨져있습니다 . res = ['aaa','bbb','ccc']
      // 이제 이것들을 출력해줍니다.
      this.subjects.forEach(subject => {
        res.split("#").forEach(async token => {
          let data = await FirebaseService.getPost(token, subject);
          if (data) {
            // data.id는 토큰 ,subject는 주제입니다.
            data.id = token;
            data.subject = subject
            this.posts.push(data);
          }
        });
      });
      console.log(this.posts, "zzzz");
    },
    setContent() {
      this.msg = "해당 내용이 포함된 게시글을 보여드립니다.";
      this.posts = [];

      this.subjects.forEach(async subject => {
        const res1 = await FirebaseService.getPostTC(subject);
        res1.forEach(post => {
          if (post.content.includes(this.header)){
            post.subject = subject
            this.posts.push(post)
          }
        })
      })
    },

    setTitle() {
      this.msg = "해당 제목이 포함된 게시글을 보여드립니다.";
      this.posts = [];

      this.subjects.forEach(async subject => {
        const res1 = await FirebaseService.getPostTC(subject);
        res1.forEach(post => {
          if (post.title.includes(this.header)){
            post.subject = subject
            this.posts.push(post)
          }
        })
      })
    }
  }
};
</script>

<style scoped>
.post-time {
  border-top: 2px dotted #e0e0e0;
  border-bottom: 2px dotted #e0e0e0;
  position: relative;
  top: 8%;
}

.post-con {
  display: flex;
}
.post-con-left {
  float: left;
  width: 100%;
}

@media screen and (max-width: 1000px) {
  .post-con-right {
    transform: scale(0.7, 0.7);
  }
}
</style>
