<template>
  <li class="flex items-center justify-between p-3 pl-6 transition duration-300 cursor-pointer hover:bg-gray-500">
          <div class='w-2/6'>
            <button type="button" class="z-50 w-8 h-8 text-xl text-black bg-white rounded-full focus:outline-none" 
          @click.prevent="newSong(song)">
            <i class="fas fa-play"
            :class="{'fa-play': !playing, 'fa-pause': playing}"></i>
          </button>
            <router-link :to="{name:'Song',params:{id:song.docID}}" class="ml-4 -mt-4 font-bold text-white">{{song.modified_name}}</router-link>
          </div>
          <div class="text-lg text-left text-white ">
            <!-- <span class="">{{ $t('song.uploadedby') }} {{song.display_name}}</span> -->
            <router-link :to="{name:'Uploader',params:{pp:song.display_name}}" class="ml-32 -mt-4 text-sm font-bold text-white">{{ $t('song.uploadedby') }} {{song.display_name}}</router-link>
          </div>
          <div class="flex justify-between w-1/5 text-lg text-white">
            <router-link custom 
            :to="{name:'Song',params:{id:song.docID}, hash:'#comments'}"
            v-slot="{navigate}">
              <span class="comments" @click="navigate">
                
                {{song.comment_count}}
              </span>
            </router-link>
            <router-link custom 
            :to="{name:'Song',params:{id:song.docID}, hash:'#comments'}"
            v-slot="{navigate}">
              <span class="" @click="navigate">
                <!-- <i class="text-gray-600 fa fa-comments"></i> -->
                <!-- <i class="fas fa-stream"></i> -->
               {{ duration }}
              </span>
            </router-link>
          </div>
    </li>
</template>

<script>
import {mapActions, mapState,mapGetters} from 'vuex';
export default {
    props:["song"],
    methods:{
      ...mapActions(['newSong']),   
    },
    computed: {
        ...mapGetters(['playing']),
        ...mapState({
      duration: (state) => state.player.duration,
    }),
    }
}

</script>

<style>

</style>