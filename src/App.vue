<template>
  <div>
    <svg
        class="cnv_tracks"
    ref="canvas" >
      <path v-for="track in tracksArray" :key="track.id"
            :d="`M ${track.path.startX},${track.path.startY}
             C ${track.path.endX} ${track.path.startY},${track.path.startX} ${track.path.endY},${track.path.endX} ${track.path.endY},`"
            stroke="#7555f6" stroke-width="3" stroke-linecap="round" fill="none"></path>
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
      limit:6,
      test:[]
    }
  },
  mounted(){
    axios.get('https://functions.yandexcloud.net/d4e7argpi2fe7epuh65b').then(response=>{
      this.test=response.data
      this.getNodes( this.test,1,100,200);
    })
  },
  methods:{
    getNodes(nodeList,indexOfZ,paddingX=0,paddingY=0,brothers=1,id='0'){
      let lineShift=0;
      let gapShift=0;
      nodeList.forEach((itm,index)=>{
        let children = itm.nodes?.length || 0
        if(children>this.limit){
          children=this.limit
        }
        if(children>5){
          lineShift += (children - 5) * 18.4
        }
        if(index<(this.limit)) {
          let newId = id;
          let top = paddingY - (200) * (index) + 150 * 0.5-lineShift
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
            if(brothers<=5){
              gapShift=(5-(brothers))*18.4/2
            }
            //console.log(lineShift,newId,children)
            this.tracksArray.push({
              id: newId,
              path: {
                startX: paddingX - 60,
                startY:paddingY+20+(92)-index*18.4-gapShift,//-1.05*(shift)+paddingY-18*index+(brothers + 1)*(2*(brothers + 1)+0.5);
                endX: paddingX + 25,
                endY:top+75+(lineShift*0.5),
              },
            })
          }
          if (itm.nodes) {
            this.getNodes(itm.nodes, ++indexOfZ, paddingX + 300, top+lineShift, children, newId);
          }
        }
      })
    },
    moveTrack(e,track,shiftx,shifty,outTracks,blockHeight){
      let target=this.tracksArray.find((itm)=>{return itm.id==track});
      if(target) {
        target.path.endX = e.pageX - shiftx;
        target.path.endY = e.pageY - shifty+75+0.5*(blockHeight-92);
      }
        let gap=outTracks.length<5?((5-outTracks.length)/2):0
       if(outTracks){
         outTracks.forEach((trackId,index)=>{
           let target=this.tracksArray.find((itm)=>{return itm.id==trackId});
           target.path.startX=e.pageX-shiftx+218;
           target.path.startY= e.pageY - shifty+blockHeight+19-(index)*18.4-gap*18.4
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
  height: 100vh;
  width:100vw;
 /* border: 2px solid red;*/
  z-index: 0;
}
</style>
