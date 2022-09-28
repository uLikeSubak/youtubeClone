<template>
  <v-container id="Watch" class="fill-height" fluid>
    <div class="watchSection fill-height">
      <div class="watchContainer">
        <v-skeleton-loader large tile :loading="videoLoading">
          <v-responsive>
            <video controls style="height: 100%; width: 100%">
              <source :src="videoUrl !== null && videoUrl" type="video/mp4" />
            </video>
          </v-responsive>
        </v-skeleton-loader>

        <div class="videoInfo">
          <h2 class="videoTitle">{{ watchVideo.title }} - {{ watchVideo.description }}
            <!-- 좋아요 누르기!  -->
          <v-btn @click="createFeeling(watchVideo.id)" >
            <v-icon v-if="isChooseFeeling === false && clickedFeelingVideoId === watchVideo.id ">mdi-thumb-up</v-icon>
            <v-icon v-else color="blue">mdi-thumb-up</v-icon>
            <div>{{ watchVideo.likes }} </div>
            </v-btn>
          <v-btn><v-icon>mdi-thumb-down</v-icon></v-btn>
          </h2> 
          <h3 class="videoTitle">{{ watchVideo.userId.channelName }}</h3>
  

        </div>
      </div>
    </div>

    <div class="listSection fill-height">
      <div class="listContainer">
        <VideoListCard
          v-for="video in videos"
          :key="video.id"
          :video="video"
          :channel="video.userId"></VideoListCard>
      </div>
    </div>
    <div class="commentTitle" >
      <h2> 댓글 </h2>
        <div v-for="(comment, index) in comments" :key="index">
          <!-- 댓글 -->
          <div>
            <span>{{ comment.userId.channelName }} : {{ comment.text }}</span>
              <v-btn @click="replyInputStatus(comment)" class="mx-2" fab dark small color="blue">쓰기
              </v-btn>
            <!-- 대댓글 버튼 -->
              <v-btn @click="getReplies(comment)" class="mx-2" fab dark small color="grey">보기
              </v-btn>
              <v-btn @click="loadComment(comment.text, comment.id)" class="mx-2" fab dark small color="orange">수정
              </v-btn>
              <v-btn @click="deleteComment(comment.id)" class="mx-2" fab dark small color="red"> 삭제
              </v-btn>
          </div>

          <!-- 대댓글 보기 -->
            <div v-if="recommentListStatus === true && comment.id === clickedComment.id" :id="comment.id">
              <div v-for="(reply, index) in comment.replies" :key="index">
                <div>{{ reply.userId.channelName}} : {{reply.text}}</div>
                  <v-btn @click="loadUpdateReplyInput(reply)" class="mx-2" fab dark small color="orange">수정
                  </v-btn>
                  <v-btn @click="deleteReply(reply.id)" class="mx-2" fab dark small color="red"> 삭제
                  </v-btn>
                  <br />
                   <!-- 대댓글 수정 -->
                    <div v-if="recommentUpdateStatus === true && reply.id === clickedReply.id">
                      <div>  
                      <ValidationProvider
                       name="대댓글 수정"                                                                                                                                                                                                                              
                      rules="required|min:3|max:100"
                      v-slot="{ errors }">
                      <v-text-field
                      :error-messages="errors"       
                      label="수정할 내용 입력" 
                      v-model="formData.reply">
                      </v-text-field>
                    </ValidationProvider>
                      <v-btn @click="updateReply(reply)" depressed block color="blue" class="white--text">대댓글
                      </v-btn>
                      </div>
                    </div>
              </div>  
            </div>
          <!-- 대댓글 작성 -->
            <div v-if="recommentStatus === true && comment.id === clickedComment.id">
              <ValidationProvider
                name="대댓글"                                                                                                                                                                                                                              
                rules="required|min:3|max:100"
                v-slot="{ errors }">
                  <v-text-field
                  :error-messages="errors"       
                  label="대댓글 입력" 
                  v-model="formData.reply">
                  </v-text-field>
              </ValidationProvider>
                <v-btn @click="createReply(comment.id)" depressed block color="blue" class="white--text">대댓글
                </v-btn>
            </div>
        </div>
    </div>
    <br />
    <div>
      <h3> 댓글 쓰기 </h3>
      <ValidationProvider
      name="댓글"
      rules="required|min:3|max:100"
      v-slot="{ errors }">
      <v-text-field 
      :error-messages="errors"
       label="댓글 입력" 
       v-model="formData.comment">
       </v-text-field>
      </ValidationProvider>
      <br />
      <v-btn
      depressed
      block
      color="blue"
      class="white--text"
      @click="createComment"
      >댓글</v-btn>
    </div>
  </v-container>

</template>
<script>
import axios from 'axios';
import Validate from '@/mixins/Validate.vue';
import VideoListCard from '@/components/VideoListCard.vue';


export default {
  name: 'Watch',
  data: () => ({
    GE_USER_DATA: JSON.parse(localStorage.getItem('user')) || null,

    updatedCommentId:'',

    formData:{
      comment: '',
      videoId:'',
      userId:'',
      reply:'',
    },
    videoCount: '',
    replyId: [],
    replies: [],
    relplyArr: [],
    userId: '',
    comments: [],
    videos: [],
    watchVideo: {},
    videoUrl: null,
    videoLoading: false,

    recommentStatus: false,
    recommentListStatus: false,
    recommentUpdateStatus: false,
    isChooseFeeling: false,

    clickedFeelingVideoId:'',
    clickedComment: {},
    clickedReply: {},
  }),
  components: {
    VideoListCard,
  },
  watch: {
    $route(to, from) {
      if (to.path != from.path) {
        this.getWatchData(this.$route.params.id);
        this.getVideoComments(this.$route.params.id);
        this.checkFeeling(this.$route.params.id);
        console.log('this.$route.params.id',this.$route.params.id)
      }
    },
    watchVideo() {
      this.videoUrl = `${process.env.VUE_APP_URL}/uploads/videos/${this.watchVideo.url}`;
      console.log('this.videoUrl',this.videoUrl);
    },
    
  },
  methods: {
    async getWatchData(id) {
      console.log("this.$route.params.id",this.$route.params.id);
      console.log('id',id)
      this.videoLoading = true;
      this.watchVideo = {};

      await axios
        .get(process.env.VUE_APP_API + `/videos/${id}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          console.log(
            'getWatchData - response : ', response.data.data);
          this.watchVideo = response.data.data;
          console.log("this.watchVideo",this.watchVideo)
          this.videoLoading = false;
        })
        .catch((error) => {
          console.log('getWatchData - error : ', error);
        });
    },
    async getVideos() {
      await axios
        .get(process.env.VUE_APP_API + '/videos/public', {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          console.log('getVideos - response : ', response, response.data.data);
          this.videos = response.data.data;
        })
        .catch((error) => {
          console.log('getVideos - error : ', error);
        });
    },

    async getVideoComments(){
      await axios
      .get(process.env.VUE_APP_API + `/comments/${this.$route.params.id}/videos`,{
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=> {
        console.log('getVideoComment - response', response);
        this.comments = response.data.data;
        console.log('this.comments',this.comments); 

      })
      .catch((error)=>{
        console.log('getVideoComment - error ', error);
      })
    },


    async createComment(){
      if(this.updatedCommentId){
        const commentDataForm = new FormData()
        commentDataForm.append('text', this.formData.comment)
        await axios
        .put(process.env.VUE_APP_API + `/comments/${this.updatedCommentId}`, commentDataForm,{
            headers:{
              Authorization: `Bearer ${localStorage.getItem('token')}`
            }
        })
        .then((response)=>{
          console.log('updateComment - response ', response);
          this.getVideoComments(this.$route.params.id);
          this.updatedCommentId = '';
          this.formData.comment = '';
        })
        .catch((error)=> {
          console.log('pdateComment - error', error);
        })
      }else{
      const commentDataForm = new FormData()
      commentDataForm.append('text', this.formData.comment)
      commentDataForm.append('videoId', this.watchVideo.id)
      commentDataForm.append('userId', this.GE_USER_DATA.userId)
      await axios
      .post(process.env.VUE_APP_API + '/comments', commentDataForm,{
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=> {
        console.log("createComment - response ", response);
        this.getVideoComments(this.$route.params.id)
        this.formData.comment = '';
      })
      .catch((error)=> {
        console.log('createComment - error ', error);
      })
      }  
    },

    async deleteComment(comment){
      await axios
      .delete(process.env.VUE_APP_API + `/comments/${comment}`,{
        headers:{
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=> {
        console.log('deleteComment - response', response);
        this.getVideoComments(this.$route.params.id);
      })
      .catch((error)=>{
        console.log('deleteComment - error', error);
      })
    },
    async loadComment(comment, updatedComment) {
      this.formData.comment = comment;
      this.updatedCommentId = updatedComment;
      console.log(this.formData.comment, this.updatedCommentId)
      // this.inputStatus = true;
    },

    async mountTest(){
			console.log('mount 성공!')
		},

    async replyInputStatus(comment){
      if(this.recommentStatus === true){
        this.recommentStatus = false;
      }else{
        this.clickedComment = comment;
        this.recommentStatus = true;   
      }
    },

    async createReply(commentId){
      const replyFormData = new FormData();
      replyFormData.append('text', this.formData.reply);
      replyFormData.append('commentId', commentId);
      replyFormData.append('userId', this.GE_USER_DATA.id);

      await axios
      .post(process.env.VUE_APP_API + '/replies', replyFormData ,{
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=>{
        console.log( 'createReply - response', response)
        this.getVideoComments(this.$route.params.id)
        this.formData.reply = '';
      })
      .catch((error)=>{
        console.log(' createReply - error', error)
      })

    },

    async getReplies(comment){
      console.log("before if ",this.recommentListStatus)
      if(this.recommentListStatus === true){
        this.recommentListStatus = false;
        console.log("inside if ",this.recommentListStatus)
      }else{
      console.log('comment : ', comment)
      this.clickedComment = comment;

      await axios
      .get(process.env.VUE_APP_API + '/replies',{
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=>{
        console.log("inside else - response",this.recommentListStatus)
        this.recommentListStatus = true;
        console.log("inside else - response",this.recommentListStatus)
        console.log('getReplies - response', response);
        this.replies = response.data.data;
        console.log("this.replies",this.replies);
        this.getVideoComments(this.$route.params.id)
      })
      .catch((error)=>{
        console.log('getReplies - error', error)
      })
      } 
    },

    async deleteReply(reply) {
      await axios
      .delete(process.env.VUE_APP_API + `/replies/${reply}`,{
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=>{
        console.log('deleteReplay - response', response)
        this.getVideoComments(this.$route.params.id);
      })
      .catch((error)=>{
        console.log('deleteReplay - error', error)
      })
    },


    async loadUpdateReplyInput(reply){
      console.log("reply.id",reply.id)
      console.log("loadUpdateReplyInput - response",this.recommentUpdateStatus)
      if(this.recommentUpdateStatus === true){
        this.recommentUpdateStatus = false;
      }else{
        this.recommentUpdateStatus = true;
        this.clickedReply = reply;
      }
      console.log(this.recommentUpdateStatus);
      console.log(this.clickedReply);
    },

    
    async updateReply(reply){
      console.log("updateReply - reply",reply.id)
      const updateReplyFormData = new FormData()
      updateReplyFormData.append('text', this.formData.reply)
      await axios
      .put(process.env.VUE_APP_API + `/replies/${reply.id}`,updateReplyFormData,{
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=> {
        console.log(' updateReply - response',response);
        this.formData.reply = "";
        this.getVideoComments(this.$route.params.id);
      })
      .catch((error)=> {
        console.log(' updateReply - error',error);
      })
    },

    async createFeeling(videoId) {    
      const feelingFormData = new FormData();
      feelingFormData.append('type', "like")
      feelingFormData.append('videoId', videoId)
      feelingFormData.append('userId', this.GE_USER_DATA.id)

      await axios
      .post(process.env.VUE_APP_API + '/feelings', feelingFormData, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=>{
        console.log('createFeeling - response', response)
        if(this.isChooseFeeling === true){
          this.isChooseFeeling = false;
          this.checkFeeling();
        }else{
        this.isChooseFeeling = true;
        this.clickedFeelingVideoId = videoId 
        this.checkFeeling();
        }
      })
      .catch((error)=>{
        console.log('createFeeling - error', error)
      })

    },

    async checkFeeling(url){
      console.log("url",url);
      const checkFeelingFormData = new FormData();
      checkFeelingFormData.append('videoId', this.watchVideo.id || url) // append할 때 벨류로  a||b 로 a에 값이 없으면 b/
      checkFeelingFormData.append('userId', this.GE_USER_DATA.id) // a에 값이 있으면 a/로 표현해서 전에 값이 남아 있으면 전에 값을 보여주게 한다?
      console.log("checkFeelingFormData",checkFeelingFormData);
      await axios
      .post(process.env.VUE_APP_API + '/feelings/check', checkFeelingFormData, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      })
      .then((response)=>{
        console.log('checkFeeling - response', response)
        console.log("this.isChooseFeeling", this.isChooseFeeling )
        this.getWatchData(this.$route.params.id);
      })
      .catch((error)=>{
        console.log('checkFeeling - error', error);
      })
    },


  },
  mounted() {
    this.getVideos();
    this.getWatchData(this.$route.params.id);
    this.getVideoComments(this.$route.params.id);
    this.mountTest('mount 테스트!');

  },
  mixins: [Validate],

};
</script>
<style>
#Watch {
  display: grid;
  grid-template-columns: 70% 30%;
}

.videoInfo {
  box-sizing: border-box;
  padding: 10px;
  width: 100%;
}

.listSection {
  align-items: center;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  padding-left: 10px;
  width: 100%;
}
.listContainer {
  align-items: center;
  box-sizing: border-box;
  row-gap: 5px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  padding-left: 10px;
  width: 100%;
}
.commentTitle{
  font-size: 15px;  
}
</style>
