<template>
  <main>
     <!-- Music Header -->
    <section class="relative w-full mb-8 text-center text-white py-14">
        <div class="box-border absolute inset-0 w-full h-full bg-cover music-bg"
          style="background-image: url(/assets/img/song-header.png)">
        </div>
        <div class="container flex items-center mx-auto">
          <!-- Play/Pause Button -->
          <button type="button" class="z-50 w-24 h-24 text-3xl text-black bg-white rounded-full focus:outline-none" 
          @click.prevent="newSong(song)">
            <i class="fas fa-play"
            :class="{'fa-play': !playing, 'fa-pause': playing}"></i>
          </button>
          <div  class="z-50 ml-8 text-left">
            <!-- Song Info -->
            <div class="text-3xl font-bold">{{song.modified_name}}</div>
            <div v-if="song.genre">
              <router-link :to="{ name: 'Genre', params: { tags: song.genre }}" class="-mt-4 text-sm font-bold text-white ">{{song.genre}}</router-link>
            </div>
            <div v-else class="text-white">
              Loading
            </div>
          </div>
        </div>
      </section>
      <!-- Form -->
      <section class="container mx-auto mt-6" id="comments">
        <div class="relative flex flex-col rounded">
          <div class="px-6 pt-6 pb-5 font-bold border-b border-gray-200">
            <!-- Comment Count -->
            <span class="text-white card-title">{{ $tc('song.comment_count', song.comment_count, {
                count: song.comment_count 
              }) 
              }}</span>
            <i class="float-right text-2xl text-green-400 fa fa-comment-alt"></i>
          </div>
          <div class="p-6">
            <div class="p-4 mb-4 font-bold text-center text-white" v-if="comment_show_alert" :class="comment_alert_variant">
              {{comment_alert_message}}
            </div>
            <vee-form :validation-schema="schema" @submit="addComment"
            v-if="userLoggedIn">
              <vee-field as="textarea" name="comment"
                class="block w-full py-1.5 px-3 text-gray-200  rounded-b-lg  transition
                  duration-500 focus:outline-none focus:border-black rounded mb-4 bg-gray-700" 
                placeholder="Your comment here..."></vee-field>
                <ErrorMessage class="block text-red-600" name="comment"></ErrorMessage>
              <button type="submit" class="float-right py-1.5 px-3 rounded text-white bg-green-600"
              :disabled="comment_in_submission">{{ $t('submit') }}</button>
            </vee-form>
            <!-- Comment Sorting -->
            <select v-model = "sort"
              class="block mt-4 py-1.5 px-3 bg-green-600 text-white border border-gray-300 transition rounded
              duration-500 focus:outline-none focus:border-black">
              <option value="1">{{ $t('comment.latest') }}</option>
              <option value="2">{{ $t('comment.newest') }}</option>
            </select>
          </div>
        </div>
      </section>
      <!-- Comments -->
      <ul class="container mx-auto">
        <li class="p-6 border-b border-gray-500"
        v-for="comment in formattedComments " :key="comment.docID">
          <!-- Comment Author -->
          <div class="mb-5">
            <div class="font-bold text-white">{{comment.name}}</div>
            <time class="text-white">{{comment.datePosted}}</time>
          </div>

          <p class="text-white">
            {{comment.content}}
          </p>
        </li>
      </ul>
  </main>
</template>

<script>
import {songsCollection, auth, commentsCollection} from '@/includes/firebase';
import {mapState, mapActions} from 'vuex';
import {formatDistanceToNow} from "date-fns"
export default {
  name: "Song",
  data() {
    return{
      song:{},
      schema: {
        comment: 'required|min:3',
      },
      comment_in_submission: false,
      comment_show_alert:false,
      comment_alert_variant:'',
      comment_alert_message: '',
      comments:[],
      sort: '1',
    };
  },
  computed: {
     ...mapState({
      userLoggedIn: (state) => state.auth.userLoggedIn,
    }),
    // sortedComments(){
    //   return this.comments.slice().sort((a,b) => {
    //     if(this.sort === '1'){
    //       return new Date(b.datePosted) - new Date(a.datePosted);
    //     }
    //     return new Date(a.datePosted) - new Date(b.datePosted);
    //   });
    // },
    formattedComments(){
      let formatCom =  this.comments.map((com)=>{
            let formatTime=  formatDistanceToNow(new Date(com.datePosted))
            return  {...com,datePosted:formatTime}// 
        })// [{}]
        return formatCom.slice().sort((a,b) => {
          if(this.sort === '1'){
            return b.datePosted < a.datePosted;
          }
        return a.datePosted > b.datePosted;
      });
      }
  },
  async beforeRouteEnter(to, from, next) {
    const docSnapshot = await songsCollection.doc(to.params.id).get();

    next((vm) => {
      if (!docSnapshot.exists) {
      vm.$router.push({ name: 'home' });
      return;
    } 

    const { sort } = vm.$route.query;
    vm.sort = sort === '1' || sort === '2' ? sort : '1';

    vm.song =  docSnapshot.data();
    vm.getComments();
    });

  },
  methods: {
    ...mapActions(['newSong']),
    async addComment(values, {resetForm}) {
      this.comment_in_submission = true;
      this.comment_show_alert = true;
      this.comment_alert_variant = "bg-blue-500";
      this.comment_alert_message = 'Please wait! Your comment is being submitted';

      const comment = {
        content: values.comment,
        datePosted: new Date().toString(),
        sid : this.$route.params.id, 
        name:auth.currentUser.displayName,
        uid: auth.currentUser.uid,
      };
      await commentsCollection.add(comment);
      
      this.song.comment_count += 1;
      await songsCollection.doc(this.$route.params.id).update({
        comment_count: this.song.comment_count,
      });

      this.getComments();

      this.comment_in_submission = false;
      this.comment_alert_variant = "bg-green-500";
      this.comment_alert_message = 'Your comment is added!';

      resetForm();
    },
    async getComments(){
      const snapshots = await commentsCollection
      .where('sid', '==', this.$route.params.id,)
      .orderBy("datePosted")
      .get();

      this.comments = [];
      
      snapshots.forEach((doc) => [
        this.comments.push({
          docID: doc.id,
          ...doc.data(),
        }),
      ]);
    },  
  },
  watch: {
    sort(newVal){
      console.log(this.sort)
      if(newVal === this.$route.query.sort){
        return;
      }
      this.$router.push({
        query:{
          sort:newVal,
        }
      })
    }
  },
};
</script>