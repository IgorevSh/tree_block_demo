<template>
<div ref="block" @mousedown="mouseDown" v-on:mouseup="eventRemover" class="block_wrapper" :style="{left:marginLeft+'px',top:marginTop+'px',zIndex:node.zIndex,}">
  <div class="block_header">
    <span>{{node.title}}</span>
  </div>
  <div class="block_body" ref="body">
    <div class="in_dots_list">
      <div v-if="node.hasParent" class="in_connect">
        <div class="dot"></div>in
      </div>
    </div>
    <div class="block_content"></div>
    <div class="out_dots_list">
      <div v-for="n in node.children" :key="n" class="in_connect">
         out <div ref="out_dot" class="dot"></div>
      </div>
    </div>
  </div>
</div>
</template>

<script>
export default {
  name: "BlockItem",
  props:{
    node:{
      type: Object,
    },
    canvas:{type: Object}
  },
  data(){
    return{
      shiftX:0,
      shiftY:0,
      marginTop:0,
      marginLeft:0,
    }
  },
  mounted(){
    this.marginTop=this.node.paddingTop;
    this.marginLeft=this.node.paddingLeft;

  },
  methods:{
    mouseDown(event){
      let node= this.$refs.block;
      this.shiftX = event.clientX - node.getBoundingClientRect().left;
      this.shiftY = event.clientY - node.getBoundingClientRect().top;
      this.moveNode(event.pageX, event.pageY);
      //if(this.stableNode()) {
        document.addEventListener('mousemove', this.onMouseMove, false);
     // }
    },
    moveNode(moveX,moveY){
     // if(this.stableNode()){
      this.marginLeft=moveX-this.shiftX
      this.marginTop=moveY-this.shiftY
//    }
    },
    onMouseMove(event){
     this.moveNode(event.pageX, event.pageY);
     let track =this.node.trackId
     let children= this.node.children;
      let tracksOut=[];
     for (let i =0;i<children;i++){
       tracksOut.push(track+'.'+i);
     }
    //  if(this.stableNode()){
        this.$emit('moveTrack', event, track, this.shiftX - 25, this.shiftY, tracksOut,this.$refs.body.offsetHeight);
    //  }
    },
    stableNode(){
      let svgSize=this.canvas.getBoundingClientRect();
      let nodeSize=this.$refs.block.getBoundingClientRect();

      if(svgSize.bottom<nodeSize.bottom){
        this.marginTop=svgSize.bottom-120;
        //this.eventRemover();
       return false;
      }
      else if(svgSize.top>nodeSize.top){
        this.marginTop=svgSize.top+1;
        //this.eventRemover();
        return false;
      }
      else if(svgSize.left>nodeSize.left){
        this.marginLeft=svgSize.left+1;
       // this.eventRemover();
        return false;
      }
      else if(svgSize.right<nodeSize.right){
        this.marginLeft=svgSize.right-235;
      //  this.eventRemover();
        return false;
      }else{
        return true;
      }

    },
    eventRemover(){
      document.removeEventListener('mousemove', this.onMouseMove,false);
    }
  }
}
</script>

<style scoped lang="scss">
.block_wrapper{
  width: 250px;
  position: absolute;
  left: 20px;
  top: 20px;
  //border: 1px solid black;
  border-radius:5px 5px 0 0;
  box-shadow: 0 2px 10px -5px black;
  &:hover{
    cursor: pointer;
  }
  .block_header{
    border-radius:5px 5px 0 0;
    padding: 5px 10px;
    background-color: #7555f6;
    color:white
  }
  .block_body{
    min-height: 92px;
    display: flex;
   // background-color: white;
    .in_dots_list,.out_dots_list{
      width: 20%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      min-height: 100%;
      padding: 0 5px ;
      .in_connect{
        display: flex;
        align-content: center;
        align-items: center;
        justify-content: space-between;
        gap: 5px;
      }
      .dot{
        width: 8px;
        height: 8px;
        border: 2px solid #7555f6;
        border-radius: 50%;
        background-color: #b9abf8;
      }
    }
    .in_dots_list{
     align-content: flex-end;
     align-items: flex-end;
    }
    .block_content{
      width: 60%;
    }
  }
}
</style>
