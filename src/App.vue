<template>
  <div>
    <svg
        class="cnv_tracks"
    ref="canvas" :style="{height:minHeight+'px',width:minWidth+'px'}">
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
      limit:10,
      maxTop:0,
      minWidth:0,
      minHeight:0,
      test:[]
    }
  },
  mounted(){
    axios.get('https://functions.yandexcloud.net/d4e7argpi2fe7epuh65b?').then(response=>{
      this.minHeight=0
      this.minWidth=window.innerWidth
      this.test=response.data
      this.getNodes(this.test,1,100,0);
      if(this.maxTop<0){
        this.shiftGraph()
      }
    })
  },
  methods:{
    getNodes(nodeList,indexOfZ,paddingX=0,paddingY=0,brothers=1,id='0',shift=0){
      let lineShift=shift;
      let gapShift=0;
      let brotherChild=1;
      nodeList.forEach((itm,index)=>{
         console.log(brotherChild)
        let children = itm.nodes?.length || 0
        if(children>this.limit){
          children=this.limit
        }
        if(children>5){
          lineShift = (children - 5) * 18.4
        }
        if(index<(this.limit)) {
          let newId = id;
          console.log(itm.title,brotherChild)
          let top = paddingY - (130) * (index) - lineShift+(brothers*92-(brotherChild-1)*130);
          if(this.maxTop>top){
            this.maxTop=top;
          }
          if(paddingX>this.minWidth){
            this.minWidth=paddingX
          }
          if(this.minHeight<top){
            this.minHeight=top+92
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
            let childShift=0;
            let brotherShift=0;
            newId = newId + '.' + index;
            if(brothers<=5){
              gapShift=(5-(brothers))*18.4/2
            }else{
              brotherShift=(brothers-5)*18.4
            }
            if(children>5){
              childShift=(children-5)*18.4/2
            }

            this.tracksArray.push({
              id: newId,
              path: {
                startX: paddingX - 60,
                startY:paddingY+20+92+brotherShift-gapShift-18.4*index,//-1.05*(shift)+paddingY-18*index+(brothers + 1)*(2*(brothers + 1)+0.5);
                endX: paddingX + 25,
                endY:top+28+46+childShift,
              },
            })
          }
          if (itm.nodes) {
            this.getNodes(itm.nodes, ++indexOfZ, paddingX + 300,top, children, newId,lineShift);
          }
          brotherChild+=children===0?1:children;
        }
      })
    },
    shiftGraph(){
      let topShift=this.maxTop*(-1)
      this.nodesArray.forEach((itm)=>{
        itm.paddingTop+=topShift
      })
      this.tracksArray.forEach((itm)=>{
        itm.path.startY+=topShift
        itm.path.endY+=topShift
      })
      this.minHeight+=topShift+92
    },
    moveTrack(e,track,shiftx,shifty,outTracks,blockHeight){
      let target=this.tracksArray.find((itm)=>{return itm.id==track});
      if(e.pageY - shifty+blockHeight>this.minHeight){
        this.minHeight=e.pageY - shifty+blockHeight
      }
      if(e.pageX - shiftx+25>this.minWidth){
        this.minWidth=e.pageX - shiftx+25
      }
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
 /* border: 2px solid red;*/
  z-index: 0;
}
</style>
