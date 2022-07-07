<template>
  <div>
    <svg
        class="cnv_tracks"
    ref="canvas">
      <path v-for="track in tracksArray" :key="track.id"
            :d="`M ${track.path.startX},${track.path.startY}
             C ${track.path.endX} ${track.path.startY},${track.path.startX} ${track.path.endY},${track.path.endX} ${track.path.endY},`"
            stroke="#7555f6" stroke-width="5" stroke-linecap="round" fill="none"></path>
    </svg>

    <BlockItem
        v-for="node in nodesArray" :key="node.title"
        :node ="node"
        :canvas="$refs.canvas"
        @move-track="moveTrack"
        @moveBG="moveBG"
    />
    <!--<div v-for="bg in bgArray" :key="bg.id" :style="{left:bg.left+'px',top:bg.top+'px'}"  class="underblock">
      {{bg.trackId}}
    </div>-->
  </div>
</template>

<script>
import BlockItem from './components/BlockItem.vue'
import test from '../public/test.json'

export default {
  name: 'App',
  components: {
    BlockItem
  },
  data(){
    return{
      nodesArray:[],
      tracksArray:[],
     // bgArray: {},
    }
  },

  mounted(){
    this.getNodes(test,1,100,200);
  },
  methods:{
    getNodes(nodeList,indexOfZ,paddingX=0,paddingY=0,brothers=0,id='0'){
      nodeList.forEach((itm,index)=>{
        let newId=id;
        let top =paddingY-(200)*(index)+(brothers+1*150)*0.5
        this.nodesArray.push({
          title:itm.title,
          children:itm.nodes?.length||0,
          zIndex:indexOfZ,
          paddingLeft:paddingX,
          paddingTop:top,
          hasParent:indexOfZ>1,
          trackId:indexOfZ>1?newId+'.'+index:'0',
          childTrack:[],
        })
       // this.bgArray[`bg-${indexOfZ>1?newId+'.'+index:'0'}`]={id:`bg-${indexOfZ>1?newId+'.'+index:'0'}`,top:top,left:paddingX};
        if(indexOfZ>1) {
          newId=newId+'.'+index;
          this.tracksArray.push({
            id: newId,
            path: {
              startX: paddingX-60,
              startY: paddingY+80-18*index+(brothers+1)*2*(brothers+1)-3,
              endX: paddingX+25,
              endY: top+75,
            },
          })
        }
        if(itm.nodes){
          this.getNodes(itm.nodes,++indexOfZ,paddingX+300,top,itm.nodes?.length-1||0,newId);
        }
      })
    },
    moveTrack(e,track,shiftx,shifty,outTracks){
      let target=this.tracksArray.find((itm)=>{return itm.id==track});
      if(target) {
        target.path.endX = e.pageX - shiftx;
        target.path.endY = e.pageY - shifty;
      }
      if(outTracks){
        outTracks.forEach((itmTrack,indx)=>{
          let target=this.tracksArray.find((itm)=>{return itm.id==itmTrack});
          target.path.startX=e.pageX-shiftx+220;
          target.path.startY=e.pageY-18*indx-shifty+outTracks.length*(2*outTracks.length)-2;
        })
      }
    },
 /*
    moveBG(id,top,left){
      console.log(id,this.$refs[`bg-${id}`])
    this.bgArray[`bg-${id}`].top=top
    this.bgArray[`bg-${id}`].left=left
    }
   */

  }
}
</script>

<style>
*{
  padding: 0;
  margin: 0;
}
.cnv_tracks{
  position: absolute;
  height: 100vh;
  width:100vw;
  border: 2px solid red;
  z-index: 0;
}
.underblock{
  height: 128px;
  width: 250px;
  background-color: white;
  position: absolute;
  z-index: -1;
  border-radius:5px 5px 0 0;
  box-shadow: 0 2px 10px -5px black;
}
</style>
