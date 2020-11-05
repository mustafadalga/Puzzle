<template>
  <div class="container">
    <div class="game-status-wrap">
      <div class="game-status">
        <h5>Level</h5>
        <span >{{  level.no }}</span>
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
      <div class="puzzle" :class="isGameCompleted ? 'game-completed' : ''">
        <transition-group name="flip-list">
        <template  v-for="cell in cells" :key="cell.index">
          <div class="cell" ref="cell" :class="cell.class" :data-index="cell.index" :data-row="cell.row" :data-col="cell.col" :style="setflexBasis" @click="selectCell($event)">
            <img :src="cell.src">
          </div>
        </template>
        </transition-group>
      </div>

    </div>
    <modal :class="modalStatus ? 'open':''" :level="level" :totalLevel="levels.length" @modalToggle="modalToggle" @restartGame="restartGame" @increaseLevel="increaseLevel"/>

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
        no:1,
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
      modalStatus:false,
      isGameCompleted:false
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
  //  this.startTimer();
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
      this.level={...this.level, ...this.levels[this.level.no-1]}
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
      shuffleList=[...Array(this.level.cellLength).keys()]
      let a=shuffleList[this.level.cellLength-1]
      let b=shuffleList[this.level.cellLength-2]
      shuffleList[this.level.cellLength-1]=b
      shuffleList[this.level.cellLength-2]=a
      console.log(shuffleList)
      return shuffleList
      //return JSON.stringify(shuffleList)===JSON.stringify(this.correctCellSorting) ? this.createShuffleList() : shuffleList
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
      return this.getImgUrl(`level${(this.level.no)+'/picture'+this.level.imgIndex+'/'+(piece+1)}.jpg`);
    },
    selectCell(event){
      let cell=event.target.parentNode
      if (!this.checkCellLocationDiff(cell)) return;
      this.swapCellIndex(cell)
      this.level.movesCount++;
    },
    getEmptyCell(){
      return [...document.querySelectorAll('.cell')].find(cell => cell.classList.contains('empty'))
    },
    getCellPosition(cell){
      return [cell.getAttribute('data-row'),cell.getAttribute('data-col')];
    },
    getCellIndexForArray(cell){
      let attrIndex=this.getCellIndexForImage(cell)
      return this.cells.findIndex(item=>item.index===attrIndex)
    },
    getCellIndexForImage(cell){
      return parseInt(cell.getAttribute('data-index'));
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
      let activeCellIndex=this.getCellIndexForArray(activeCell)
      let emptyCellIndex=this.getCellIndexForArray(this.getEmptyCell())

      this.cells[activeCellIndex].index=this.getCellIndexForImage(this.getEmptyCell())
      this.cells[emptyCellIndex].index=this.getCellIndexForImage(activeCell)
      this.cells[emptyCellIndex].src=this.cells[activeCellIndex].src
      this.cells[emptyCellIndex].class=""
      this.cells[activeCellIndex].class="empty"
      this.cells[activeCellIndex].src=""
      if (this.checkCellOrder()) this.levelCompleted()
    },
    calcCellDiff(activeCell){
      let [cellRow,cellCol]=this.getCellPosition(activeCell);
      let [emptyRow,emptyCol]=this.getCellPosition(this.getEmptyCell());
      return (Math.abs(cellRow-emptyRow)===1 && Math.abs(cellCol-emptyCol)===0) || (Math.abs(cellRow-emptyRow)===0 && Math.abs(cellCol-emptyCol)===1);
    },
    checkCellLocationDiff(cell){
      return this.calcCellDiff((cell)) ? true :false;
    },
    levelCompleted(){
      clearInterval(this.timerSetInterval)
      this.isGameCompleted=true
      this.level.time=this.getTime
      setTimeout(()=>{
     //   this.modalToggle()
      },1500)
    },
    modalToggle(){
      this.modalStatus=!this.modalStatus
    },
    resetLevelData(){
      this.cells=[]
      this.level.no=this.level.no===this.levels.length ? 1 :this.level.no
      this.level.movesCount=0
      this.level.time=null
      this.imgIndex=null
      this.timer={
        second:0,
        minute:0,
        hour:0
      }
      this.isGameCompleted=false
    },
    restartGame() {
      this.resetLevelData()
      this.setLevelData()
      this.createCorrectCellOrder()
      this.setCells()
      this.modalToggle()
      this.startTimer()
    },
    increaseLevel(){
      this.resetLevelData()
      this.level.no=this.level.no+1
      this.setLevelData()
      this.createCorrectCellOrder()
      this.setCells()
      this.modalToggle()
      this.startTimer()
    }
  }
};
</script>

<style lang="scss">
@import "assets/scss/main";
.flip-list-move {
  transition: transform 300ms;
}
</style>
