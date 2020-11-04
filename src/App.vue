<template>
  <div class="container">
    <div class="game-status-wrap">
      <div class="game-status">
        <h5>Level</h5>
        <span >{{  levelNo+1 }}</span>
      </div>
      <div class="game-status">
        <h5>Moving</h5>
        <span >{{ movesCount }}</span>
      </div>
      <div class="game-status">
        <h5>Time</h5>
        <span >{{ getTime }}</span>
      </div>
    </div>
    <div class="puzzle-wrapper">
      <div class="puzzle">
        <template  v-for="cell in cells" :key="cell.index">
          <div class="cell" ref="cell" :class="cell.class" :data-src="cell.src" :data-index="cell.index" :data-row="cell.row" :data-col="cell.col" :style="setflexBasis" @click="selectCell($event)">
            <img :src="cell.src">
          </div>
        </template>
      </div>

    </div>
    <modal/>

  </div>
</template>

<script>
import levels from '@/assets/json/levels.json'
import modal from '@/components/Modal'
export default {
  name: "App",
  data(){
    return{
      cells:[],
      levels:levels,
      levelNo:0,
      levelImgCount:0,
      flexBasis:0,
      cellLength:0,
      colLength:0,
      correctCellSorting:[],
      timerSetInterval:null,
      timer: {
        second:0,
        minute:0,
        hour:0
      },
      movesCount:0,
    }
  },
  components: {
    modal
  },
  created() {
    this.setLevelData()
    this.setCells();
  },
  computed:{
    setflexBasis(){
      return {'flex-basis':'calc('+this.flexBasis+'% - .3rem)'}
    },
    getTime(){
      let second=this.timer.second > 9 ? this.timer.second : `0${this.timer.second}`
      let minute=this.timer.minute > 9 ? this.timer.minute : `0${this.timer.minute}`
      let hour=this.timer.hour > 9 ? this.timer.hour : `0${this.timer.hour}`
      return `${hour}:${minute}:${second}`
    }
  },
  mounted() {
    this.createCorrectCellOrder();
    this.startTimer();
  },
  methods:{
    startTimer(){
      this.timerSetInterval=setInterval(()=> {
        this.timer.second++
        if (this.timer.second===60){
          this.timer.second=0
          this.timer.minute++
        }
        if (this.timer.minute===60){
          this.timer.hour++;
          this.timer.minute=0
          this.timer.second=0
        }
        if (this.timer.hour===24){
          this.timer.hour=0;
          this.timer.minute=0
          this.timer.second=0
        }
      },1000)
    },
    setLevelData(){
      let level=this.levels[this.levelNo]
      this.cellLength=level.cellLength
      this.colLength=level.colLength
      this.flexBasis=level.flexBasis
      this.levelImgCount=level.imgCount
    },
    createCorrectCellOrder(){
      this.correctCellSorting=[...Array(this.cellLength).keys()]
    },
    getRandomImg(){
      return Math.floor(Math.random() * this.levelImgCount)
    },
    getImgUrl(pic) {
      return require(`./assets/img/${pic}`)
    },
    createShuffleList(){
      let shuffleList=[];
      while (shuffleList.length!==this.cellLength){
        let index = Math.floor(Math.random() * this.cellLength);
        let count = shuffleList.find(item => item === index);
        if (count==undefined){
          shuffleList.push(index);
        }
      }
      return shuffleList
    },
    setCells(){
      let row=0,col=0;
      let levelImg=this.getRandomImg()+1
       this.createShuffleList().forEach((cell,index)=>{
        this.cells[index]={
          'index':cell,
          'row':row+1,
          'col':col+1,
        }
        col++;
        if ((index+1)%this.colLength===0){
          row++;
          col=0;
        }
        if (cell===this.cellLength-1){
          this.cells[index]['class']='empty';
        }else{
          this.cells[index].src=this.getImgUrl(`level${(this.levelNo+1)+'/picture'+levelImg+'/'+(cell+1)}.jpg`);
        }
      });
    },
    selectCell(event){
      let cell=event.target.parentNode
      if (!this.checkCellLocationDiff(cell)) return;
      this.moveCard(cell)
      this.movesCount++;
    },
    getEmptyCellAttr(){
      return [this.getEmptyCell().getAttribute('data-row'),this.getEmptyCell().getAttribute('data-col')];
    },
    getEmptyCell(){
      return [...document.querySelectorAll('.cell')].find(cell => cell.classList.contains('empty'))
    },
    getActiveCellAttr(cell){
      return [cell.getAttribute('data-row'),cell.getAttribute('data-col')];
    },
    checkCellOrder(){
      let cellOrder=this.cells.map(cell=>cell.index)
      console.log(cellOrder)
      if (JSON.stringify(cellOrder) === JSON.stringify(this.correctCellSorting)){
        return true;
      }else{
        return false;
      }
    },
    swapCellIndex(activeCell){

      let activeCellNo=parseInt(activeCell.getAttribute('data-index'));
      let emptyCellNo=parseInt(this.getEmptyCell().getAttribute('data-index'));
      let activeCellIndex=this.cells.findIndex(cell=>cell.index===activeCellNo)
      let emptyCellIndex=this.cells.findIndex(cell=>cell.index===emptyCellNo)

      this.cells[activeCellIndex].index=emptyCellNo
      this.cells[emptyCellIndex].index=activeCellNo
      this.cells[emptyCellIndex].src=this.cells[activeCellIndex].src
      this.cells[emptyCellIndex].class=""
      this.cells[activeCellIndex].class="empty"
      this.cells[activeCellIndex].src=""
     // this.cells.sort((a,b) => (a.index > b.index) ? 1 : ((b.index > a.index) ? -1 : 0));
      console.log(this.checkCellOrder())
    },

    changeCellAttr(activeCell){
      this.swapCellIndex(activeCell)
    },
    moveCard(activeCell){
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
