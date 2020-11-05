<template>
  <div class="container">
    <div class="game-status-wrap">
      <div class="game-status">
        <h5>Level</h5>
        <span >{{  level.no+1 }}</span>
      </div>
      <div class="game-status">
        <h5>Moving</h5>
        <span >{{ level.movesCount }}</span>
      </div>
      <div class="game-status">
        <h5>Time</h5>
        <span >{{ getTime }}</span>
      </div>
    </div>
    <div class="puzzle-wrapper">
      <div class="puzzle">
        <transition-group name="flip-list">
        <template  v-for="cell in cells" :key="cell.index">
          <div class="cell" ref="cell" :class="cell.class" :data-src="cell.src" :data-index="cell.index" :data-row="cell.row" :data-col="cell.col" :style="setflexBasis" @click="selectCell($event)">
            <img :src="cell.src">
          </div>
        </template>
        </transition-group>
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
      level:{
        no:0,
        movesCount:0,
        imgIndex:null,
      },
      timer: {
        second:0,
        minute:0,
        hour:0
      },
      correctCellSorting:[],
      timerSetInterval:null,
    }
  },
  components: {
    modal
  },
  created() {
    this.setLevelData()
    this.createCorrectCellOrder();
    this.setCells();
  },
  computed:{
    setflexBasis(){
      return {'flex-basis':'calc('+this.level.flexBasis+'% - .3rem)'}
    },
    getTime(){
      let second=this.timer.second > 9 ? this.timer.second : `0${this.timer.second}`
      let minute=this.timer.minute > 9 ? this.timer.minute : `0${this.timer.minute}`
      let hour=this.timer.hour > 9 ? this.timer.hour : `0${this.timer.hour}`
      return `${hour}:${minute}:${second}`
    }
  },
  mounted() {
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
      this.level={...this.level, ...this.levels[this.level.no]}
      this.level.imgIndex=this.getRandomImg()+1
    },
    createCorrectCellOrder(){
      this.correctCellSorting=[...Array(this.level.cellLength).keys()]
    },
    getRandomImg(){
      return Math.floor(Math.random() * this.level.imgCount)
    },
    getImgUrl(pic) {
      return require(`./assets/img/${pic}`)
    },
    createShuffleList(){
      let shuffleList=[];
      while (shuffleList.length!==this.level.cellLength){
        let index = Math.floor(Math.random() * this.level.cellLength);
        let item = shuffleList.find(item => item === index);
        if (item===undefined){
          shuffleList.push(index);
        }
      }
      return JSON.stringify(shuffleList)===JSON.stringify(this.correctCellSorting) ? this.createShuffleList() : shuffleList
    },
    setCells(){
      let row=0,col=0;
      this.createShuffleList().forEach((cell,index)=>{
        this.cells[index]={
          'index':cell,
          'row':row+1,
          'col':col+1,
        }
        col++;
        if ((index+1)%this.level.colLength===0){
          row++;
          col=0;
        }
        if (cell===this.level.cellLength-1){
          this.cells[index]['class']='empty';
        }else{
          this.cells[index].src=this.getImgUrlFormat(cell)
        }
      });
    },
    getImgUrlFormat(piece){
      return this.getImgUrl(`level${(this.level.no+1)+'/picture'+this.level.imgIndex+'/'+(piece+1)}.jpg`);
    },
    selectCell(event){
      let cell=event.target.parentNode
      if (!this.checkCellLocationDiff(cell)) return;
      this.moveCard(cell)
      this.level.movesCount++;
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
.flip-list-move {
  transition: transform 300ms;
}
</style>
