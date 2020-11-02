<template>
  <div class="container">
    <div class="puzzle-game">
      <template  v-for="cell in cells" :key="cell.index">
        <div class="cell" :class="cell.class" :data-row="cell.row" :data-col="cell.col" :style="cell.style" @click="selectCell($event)">
        </div>
      </template>
    </div>
  </div>
</template>

<script>
import levels from '@/assets/json/levels.json'
export default {
  name: "App",
  data(){
    return{
      moveStatus:false,
      cells:[],
      levels:levels,
      levelNo:0
    }
  },
  components: {
  },
  created() {
    this.setCells();
  },
  methods:{
    getLevelImg(){
      return Math.floor(Math.random() * 6)
    },
    getImgUrl(pic) {
      return require(`./assets/img/${pic}`)
    },
    createShuffleList(){
      let shuffleList=[];
      let cellLength=this.levels[0].cellLength
      while (shuffleList.length!==cellLength){
        let index = Math.floor(Math.random() * cellLength);
        let count = shuffleList.find(item => item === index);
        if (count==undefined){
          shuffleList.push(index);
        }
      }
      return shuffleList
    },
    setCells(){
      let row=0,col=0;
      let levelImg=this.getLevelImg()+1
      this.createShuffleList().forEach((cell,index)=>{
        this.cells[index]={
          'row':row+1,
          'col':col+1,
          'style':{
            'left':`${col*8.5}rem`,
            'top':`${row*8.5}rem`,
          }
        }
        col++;
        if ((index+1)%3===0){
          row++;
          col=0;
        }
        if (cell===0){
          this.cells[index]['class']='empty';
        }else{
          this.cells[index].style.background='url('+this.getImgUrl(`level${(this.levelNo+1)+'/picture'+levelImg+'/'+(cell+1)}.jpg`)+') no-repeat center/cover';
        }
      });
    },
    whichTransitionEvent(){
      let t,el = document.createElement("fakeelement");

      let transitions = {
        "transition"      : "transitionend",
        "OTransition"     : "oTransitionEnd",
        "MozTransition"   : "transitionend",
        "WebkitTransition": "webkitTransitionEnd"
      }
      for (t in transitions){
        if (el.style[t] !== undefined){
          return transitions[t];
        }
      }
    },
    selectCell(event){
      console.log(event.target)
      if (!this.checkCellLocationDiff(event.target)) return;
      console.log(111)
      if (this.moveStatus) return ;

      this.moveStatus=true;

      event.target.addEventListener(this.whichTransitionEvent(), ()=>this.moveStatus=false);
      this.moveCard(event.target)
    },
    getEmptyCellAttr(){
      return [this.getEmptyCell().getAttribute('data-row'),this.getEmptyCell().getAttribute('data-col')];
    },
    getEmptyCellPosition(){
      let emptyCell= this.getEmptyCell()
      return [emptyCell.offsetTop,emptyCell.offsetLeft]
    },
    getEmptyCell(){
      return [...document.querySelectorAll('.cell')].find(cell => cell.classList.contains('empty'))
    },
    getActiveCellPosition(cell){
      return [cell.offsetTop,cell.offsetLeft]
    },
    getActiveCellAttr(cell){
      return [cell.getAttribute('data-row'),cell.getAttribute('data-col')];
    },
    changeCellLocation(activeCell){
      let emptyCell=this.getEmptyCell();
      let [emptyTop,emptyLeft]=this.getEmptyCellPosition();
      let [cellTop,cellLeft]=this.getActiveCellPosition(activeCell)
      activeCell.style.left=`${emptyLeft}px`;
      activeCell.style.top=`${emptyTop}px`;
      emptyCell.style.left=`${cellLeft}px`;
      emptyCell.style.top=`${cellTop}px`;
    },
    changeCellAttr(activeCell){
      let emptyCell=this.getEmptyCell()
      let [emptyRow,emptyCol]=this.getEmptyCellAttr()
      let [activeRow,activeCol]=this.getActiveCellAttr(activeCell)
      activeCell.setAttribute('data-row',emptyRow)
      activeCell.setAttribute('data-col',emptyCol)
      emptyCell.setAttribute('data-row',activeRow)
      emptyCell.setAttribute('data-col',activeCol)
    },
    moveCard(activeCell){
      this.changeCellLocation(activeCell)
      this.changeCellAttr(activeCell)
    },
    calcCellDiff(activeCell){
      let [cellRow,cellCol]=this.getActiveCellAttr(activeCell);
      let [emptyRow,emptyCol]=this.getEmptyCellAttr();
      return (Math.abs(cellRow-emptyRow)===1 && Math.abs(cellCol-emptyCol)===0) || (Math.abs(cellRow-emptyRow)===0 && Math.abs(cellCol-emptyCol)===1);
    },
    checkCellLocationDiff(cell){
      return this.calcCellDiff((cell)) ? true :false;
    },
  }
};
</script>

<style lang="scss">
@import "assets/scss/main";
</style>
