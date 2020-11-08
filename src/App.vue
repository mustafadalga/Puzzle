<template>
  <div class="container">
    <div class="game-status-wrap">
      <div class="game-status">
        <h5>Level</h5>
        <span >{{  level.no }}</span>
      </div>
      <div class="game-status">
        <h5>Moves</h5>
        <span >{{ level.movesCount }}</span>
      </div>
      <div class="game-status">
        <h5>Time</h5>
        <span >{{ getTime }}</span>
      </div>
    </div>
    <button v-if="!isGameStarted" class="btn btn-start" @click="startGame()">Start</button>
    <progressbar v-else :width="setProgressBarStatus"/>
    <div class="puzzle-wrapper">
      <div class="puzzle" :class="isGameCompleted ? 'game-completed' : ''">
        <transition-group name="flip-list">
        <template  v-for="cell in cells" :key="cell.index">
          <div class="cell" ref="cell" :class="cell.class" :data-index="cell.index" :data-row="cell.row" :data-col="cell.col" :style="setflexBasis" @click="isGameStarted ? selectCell($event) : null">
            <img :src="cell.src">
          </div>
        </template>
        </transition-group>
      </div>
    </div>
    <modal :class="modalStatus ? 'open':''" :level="level" :totalLevel="levels.length" @restartGame="restartGame" @increaseLevel="increaseLevel"/>

  </div>
</template>

<script>
import levels from '@/assets/json/levels.json'
import modal from '@/components/Modal'
import progressbar from "@/components/ProgressBar";
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
        verifiedCells:[]
      },
      timer: {
        second:0,
        minute:0,
        hour:0
      },
      correctCellSorting:[],
      timerSetInterval:null,
      modalStatus:false,
      isGameCompleted:false,
      isGameStarted:false,
    }
  },
  components: {
    progressbar,
    modal
  },
  created() {
    this.getLocalStorage()
    this.setLevelData()
    this.createCorrectCellOrder();
  },
  computed:{
    setflexBasis(){
      return {'flex-basis':'calc('+this.level.flexBasis+'% - .3rem)'}
    },
    setProgressBarStatus(){
      return (this.level.verifiedCells.length*100)/this.level.cellLength
    },
    getTime(){
      let second=this.timer.second > 9 ? this.timer.second : `0${this.timer.second}`
      let minute=this.timer.minute > 9 ? this.timer.minute : `0${this.timer.minute}`
      let hour=this.timer.hour > 9 ? this.timer.hour : `0${this.timer.hour}`
      return `${hour}:${minute}:${second}`
    }
  },
  mounted() {
    this.setCells(false);
  },
  methods:{
    startGame(){
        this.isGameStarted=true
        this.setCells();
        this.checkVerifiedAllCell()
        this.startTimer();
    },
    restartGame() {
      this.resetLevelData()
      this.setLevelData()
      this.createCorrectCellOrder()
      this.setCells(false)
      this.modalToggle()
    },
    increaseLevel(){
      this.resetLevelData()
      this.level.no=this.level.no+1
      this.setLevelData()
      this.createCorrectCellOrder()
      this.setCells(false)
      this.modalToggle()
    },
    resetLevelData(){
      this.cells=[]
      this.level.no=this.level.no===this.levels.length ? 1 :this.level.no
      this.level.movesCount=0
      this.level.time=null
      this.level.verifiedCells=[]
      this.imgIndex=null
      this.timer={
        second:0,
        minute:0,
        hour:0
      }
      this.isGameStarted=false
      this.isGameCompleted=false
    },
    setCells(shuffle_status=true){
      let row=0,col=0;
      let activeList=shuffle_status ? this.createShuffleList() : this.correctCellSorting
      activeList.forEach((cell,index)=>{
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
          this.cells[index].class='empty';
        }else{
          this.cells[index].src=this.getImgUrlFormat(cell)
        }
      });
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
    selectCell(event){
      let cell=event.target.parentNode
      if (!this.checkCellLocationDiff(cell)) return;
      this.swapCellIndex(cell)
      this.level.movesCount++;
    },
    checkCellLocationDiff(cell){
      return this.calcCellDiff((cell)) ? true :false;
    },
    calcCellDiff(activeCell){
      let [cellRow,cellCol]=this.getCellPosition(activeCell);
      let [emptyRow,emptyCol]=this.getCellPosition(this.getEmptyCell());
      return (Math.abs(cellRow-emptyRow)===1 && Math.abs(cellCol-emptyCol)===0) || (Math.abs(cellRow-emptyRow)===0 && Math.abs(cellCol-emptyCol)===1);
    },
    swapCellIndex(activeCell){
      let activeCellIndex=this.getCellIndexForArray(activeCell)
      let emptyCellIndex=this.getCellIndexForArray(this.getEmptyCell())

      this.cells[activeCellIndex].index=this.getCellIndexForImage(this.getEmptyCell())
      this.cells[emptyCellIndex].index=this.getCellIndexForImage(activeCell)

      this.cells[emptyCellIndex].src=this.cells[activeCellIndex].src
      this.checkVerifiedCell(emptyCellIndex)

      this.cells[activeCellIndex].class="empty"
      this.cells[activeCellIndex].src=""

      if (this.checkCellOrder()) this.levelCompleted()
    },
    checkCellOrder(){
      let cellOrder=this.cells.map(cell=>cell.index)
      if (JSON.stringify(cellOrder) === JSON.stringify(this.correctCellSorting)){
        return true;
      }else{
        return false;
      }
    },
    levelCompleted(){
      clearInterval(this.timerSetInterval)
      this.isGameCompleted=true
      this.level.time=this.getTime
      this.level.no===this.levels.length ? this.clearStorage() : this.setLocalStorage();
      setTimeout(()=>{
        this.modalToggle()
      },1500)
    },
    checkVerifiedAllCell(){
      this.correctCellSorting.forEach(index=>{
        if(this.cells[index].index===index){
          this.cells[index].class=this.cells[index].class ? this.cells[index].class+ ' verified-cell' : 'verified-cell'
          this.level.verifiedCells.push(index)
        }
      })
    },
    checkVerifiedCell(cellIndexForArray){
      if (this.cells[cellIndexForArray].index===this.correctCellSorting[cellIndexForArray]){
        this.cells[cellIndexForArray].class="verified-cell"
        this.level.verifiedCells.push(this.cells[cellIndexForArray].index)
      }else{
        this.cells[cellIndexForArray].class=""
        this.level.verifiedCells=this.level.verifiedCells.filter(item=>item!==this.cells[cellIndexForArray].index)
      }
      if (this.cells[this.cells.length-1].index===this.correctCellSorting[this.correctCellSorting.length-1]){
        this.level.verifiedCells.push(this.cells[this.cells.length-1].index)
      }else{
        this.level.verifiedCells=this.level.verifiedCells.filter(item=>item!==this.correctCellSorting[this.correctCellSorting.length-1])
      }
    },
    createCorrectCellOrder(){
      this.correctCellSorting=[...Array(this.level.cellLength).keys()]
    },
    getRandomImg(){
      return Math.floor(Math.random() * this.level.imgCount)
    },
    getImgUrlFormat(piece){
      return this.getImgUrl(`level${(this.level.no)+'/picture'+this.level.imgIndex+'/'+(piece+1)}.jpg`);
    },
    getImgUrl(pic) {
      return require(`./assets/img/${pic}`)
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
    getLocalStorage(){
      this.level.no = parseInt(localStorage.getItem('levelNo')) || this.level.no;
    },
    setLocalStorage(){
      localStorage.setItem('levelNo',this.level.no+1);
    },
    setLevelData(){
      this.level={...this.level, ...this.levels[this.level.no-1]}
      this.level.imgIndex=this.getRandomImg()+1
    },
    clearStorage(){
      localStorage.clear();
    },
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
    modalToggle(){
      this.modalStatus=!this.modalStatus
    },
  },
};
</script>

<style lang="scss">
@import "assets/scss/main";

</style>
