<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'app',
  components: {
    HelloWorld
  },
  created() {

    var grid = [
      ['a','x','x','x','x'],
      ['b','x','x','x','x'],
      ['c','x','x','x','x'],
      ['d','x','x','x','x'],
      ['e','x','x','x','x']
    ] //2d array
    var input = [2,4,3,1,2,2,4,1,1]
    //var input = [2,4,3,1,2]
    //var input = [1,2]

    input.forEach((block, bIndex) => { //for every item in input, check if it can fit through current level
      var levelToFitlocationsMap = []
      grid.forEach((level, gIndex) => { // for CURRENT level -> see if curr block can fit: return location (indexe(s)) of where it can fit
        var fitableLocations = this.checkFit(level,block)

        levelToFitlocationsMap.push({
          level: gIndex,
          locations: fitableLocations
        })
      })
      //find lowest level, most left justified coordinate
      var lowestRelevantLevel = null // find the level to insert block
      var longestChainEncountered = 0 // make sure block is positioned after longest chain of blocks encountered so far
      for(var i=0; i<levelToFitlocationsMap.length; i++){
        var currLevel  = i
        if(levelToFitlocationsMap[currLevel].locations.length == 0){ // level with no space to slot
          //bubble up to find location that has fitable locations
          if(currLevel >= 0){ //if not already at the top
            for(var j = currLevel-1; j>=0; j--){
              if(this.countChainLength(grid[j])>longestChainEncountered){ 
                longestChainEncountered = this.countChainLength(grid[j])
              }
            }
            for(var j = currLevel-1; j>=0; j--){
              if((levelToFitlocationsMap[j].locations.length > 0 ) && this.checkIfGreaterThanLongestChain(levelToFitlocationsMap[j].locations,longestChainEncountered-1)){
                //test each location, to make sure it can be slotted into (must not surpass longestChainEncountered))
                lowestRelevantLevel = levelToFitlocationsMap[j]
                break
              }
            }
          }
          break
        }
        if(currLevel == levelToFitlocationsMap.length-1){ //at the bottom
          lowestRelevantLevel = levelToFitlocationsMap[currLevel]
          for(var j = currLevel; j>=0; j--){
            if(this.countChainLength(grid[j])>longestChainEncountered){ 
              longestChainEncountered = this.countChainLength(grid[j])
            }
          }
        }
      }
      if(lowestRelevantLevel){
        for(var i=0; i< block; i++){
          grid[lowestRelevantLevel.level][longestChainEncountered+i] = bIndex
        }
      }
    })
        //print out grid, 
    grid.forEach(level => {
      var levelString = ''
      level.forEach(item => {
        levelString += item + ', '
      })
      console.log(levelString)
    })
  },
  methods: {
      checkIfGreaterThanLongestChain(locations, longestChainEncounteredIndex){
        //locations is array of points (array)
        //to allow an element to be added, location points must be greater than the longestChainEncounteredIndex
        var flag = true
        locations.forEach(location => {
          location.forEach(point => {
            if(point <= longestChainEncounteredIndex) {
              flag = false
            } else {
              flag = true
            }
          })
        })
        return flag
      },
      countChainLength(level){ // takes a level ARRAY e.g. ['a','x','x','x','x'] or ['1','1','1','x','x']
        //counts number of consecutive integers from the left
        var count = 0
        level.forEach( item => {
          if(!isNaN(item)) { //if it is a number
            count++
          } else {
            return count
          }
        })
        return count
      },
      checkFit(level, block) { 
        // takes a level ARRAY e.g. ['a','x','x','x','x'] or ['1','1','1','x','x']
        // and a block INT e.g. 2 => a width 2 block
        // return 'null' if block canot fit on this level at all, else
        // return all possible coordinates array where a block can fit
        var fittedCoordinates = []
        var currentPosition = []
        // find all possible coordinates of block by shifting it right across the level
        for(var i=0; i< block; i++){
          currentPosition.push(i)
        }
        while(currentPosition[currentPosition.length-1] <= level.length-1) {
            var isFit = true
            currentPosition.forEach(position => {
              if(!isNaN(level[position])){ //if the block is aleady a number
                isFit = false // cannot fit
              }
            })
            if(isFit){
              //coordinates which can fit
              fittedCoordinates.push(currentPosition)
            }
              var previousPosition = []
              currentPosition.forEach (position => {
                previousPosition.push(position)
              })
              currentPosition = []
              previousPosition.forEach(prevPos => {
                //it does not fit, increment each element in the array by 1
                currentPosition.push(parseInt(prevPos) + 1)
              })
        }
        //cannot find any coordinates that fit => return empty
        return fittedCoordinates
      },

    }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
