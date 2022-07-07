<template>
  <div>
    <svg
        class="cnv_tracks"
    ref="canvas" >
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
    />
  </div>
</template>

<script>
import BlockItem from './components/BlockItem.vue'
//import test from '/public/test.json'
import axios from 'axios'

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
      test:[]
    }
  },
  mounted(){
    axios.get('https://functions.yandexcloud.net/d4e7argpi2fe7epuh65b').then(response=>{
      this.test=response.data
      this.getNodes(this.test,1,100,1000);
    })
  },
  methods:{
    getNodes(nodeList,indexOfZ,paddingX=0,paddingY=0,brothers=0,id='0'){
      nodeList.forEach((itm,index)=>{
        if(index<4) {
          let newId = id;
          let top = paddingY - (200) * (index) + (brothers + 1 * 150) * 0.5
          let children = itm.nodes?.length || 0
            if(children>4){
              children=4
            }
          this.nodesArray.push({
            title: itm.title,
            children: children,
            zIndex: indexOfZ,
            paddingLeft: paddingX,
            paddingTop: top,
            hasParent: indexOfZ > 1,
            trackId: indexOfZ > 1 ? newId + '.' + index : '0',
            childTrack: [],
          })
          // this.bgArray[`bg-${indexOfZ>1?newId+'.'+index:'0'}`]={id:`bg-${indexOfZ>1?newId+'.'+index:'0'}`,top:top,left:paddingX};
          if (indexOfZ > 1) {
            newId = newId + '.' + index;
            this.tracksArray.push({
              id: newId,
              path: {
                startX: paddingX - 60,
                startY: paddingY + 80 - 18 * index + (brothers + 1) * 2 * (brothers + 1) - 3,
                endX: paddingX + 25,
                endY: top + 75,
              },
            })
          }
          if (itm.nodes) {
            this.getNodes(itm.nodes, ++indexOfZ, paddingX + 300, top, itm.nodes?.length - 1 || 0, newId);
          }
        }
      })
    },
    moveTrack(e,track,shiftx,shifty,outTracks,blockHeight){
      console.log(blockHeight)
      let target=this.tracksArray.find((itm)=>{return itm.id==track});
      if(target) {
        target.path.endX = e.pageX - shiftx;
        target.path.endY = e.pageY - shifty+(blockHeight-100);
      }
      if(outTracks){
        outTracks.forEach((itmTrack,indx)=>{
          let target=this.tracksArray.find((itm)=>{return itm.id==itmTrack});
          target.path.startX=e.pageX-shiftx+220;
          target.path.startY=-1*(blockHeight-100)+e.pageY-18*indx-shifty+outTracks.length*(2*outTracks.length)-2;
        })
      }
    },
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
  height: 5000px;
  width:5000px;
 /* border: 2px solid red;*/
  z-index: 0;
}
</style>
