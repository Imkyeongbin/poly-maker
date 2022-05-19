<template>
  <div class="flex">
    <section id="main">
      <div id="canvasBox" class="inline-block">
        <canvas id="canvas" :width="canvasWidth+'px'" :height="canvasHeight+'px'"></canvas>
        <!-- <img :src="image" :width="canvasWidth+'px'" :height="canvasHeight+'px'"> -->
        <img :src="image" @load="changeSize">
      </div>
    </section>
    <section id="tools">
      <div id="canvas-controller">
        캔버스
        <div>
          너비 <input type="number" v-model="canvasWidth" @change="save">px
        </div>
        <div>
          높이 <input type="number" v-model="canvasHeight" @change="save">px
        </div>

        <!-- <div class="inline-block">
          <div>
            배율 <input type="number" v-model="magnification">
          </div>
          <button @click="bigger">1/1.2배로 감소</button>
          <div class="inline-block margin5">
            <button @click="zoom(0.5)">0.5배 비율</button>
            <button @click="zoom(1)">원본 사이즈</button>
            <button @click="zoom(2)">2배 비율</button>
          </div>
          <button @click="smaller">1.2배로 증가</button>
        </div> -->
        <div>
          폴리
        </div>
        <div>
          굵기 <input type="number" v-model="polyLineWidth">
        </div>
        <div>
          이미지
        </div>
        <div>
          업로드 <input type="file" ref="imageFile" @change="uploadImage">
        </div>
        <div class="margin5 bottomPosition">
          <button @click="help">도움말</button>
        </div>
      </div>
      <div id="polyList">
        <div id="poly-controller">
          <div class="inline-block">
            <div class="margin5 inline-block">
              <button @click="addPoly">
                poly인풋추가
              </button>
            </div>
            <div class="margin5 inline-block">
              <button @click="adaptPoly">
                poly적용
              </button>
            </div>
            <div class="margin5 inline-block">
              <button @click="mergePoly">
                poly병합
              </button>
            </div>
            <div class="margin5 inline-block">
              <button @click="clearPoly">
                poly클리어
              </button>
            </div>
            <div class="margin5 inline-block">
              <button @click="clearTexts">
                텍스트클리어
              </button>
            </div>
          </div>
        </div>
        <div class="polyBox" v-for="(array, index) in polyStr" :key="'poly'+ index">
          <div class="flex center marginTop15">
            <div>
              {{ index+"." }} <br>
              <input type="checkbox" v-model="checkedPoly" :value="index">
            </div>
            <textarea v-model="polyStr[index]">
              
            </textarea>
            <Twitter v-model="polyColorJson[index]" @input="onColorChange($event, index)"></Twitter>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
<script>
import { Twitter } from 'vue-color'
export default {
  name: 'PolyMaker',
  components: {
    Twitter,
  },
  data() {
    return {
      canvasWidth : 1000,
      canvasHeight : 1000,
      image : '',
      magnification : 1,
      checkedPoly: [],
      polyStr : [[]],
      poly : [[]],
      polyColor : [[]],
      polyColorJson : [[]],
      polyLineWidth : 5,
    }
  },
  mounted(){
    this.load()  
  },
  methods:{
    smaller(){
      this.magnification = this.magnification / 1.2
    },
    zoom(value){
      this.magnification = value
    },
    bigger(){
      this.magnification = this.magnification * 1.2
    },
    changeSize(event){
      if(this.image === '' || this.image == null){
        return
      }
      this.canvasWidth = event.currentTarget.naturalWidth
      this.canvasHeight = event.currentTarget.naturalHeight
    },
    save(){
      localStorage.setItem("canvasWidth", this.canvasWidth)
      localStorage.setItem("canvasHeight", this.canvasHeight)
      localStorage.setItem("polyLineWidth", this.polyLineWidth)
    },
    // mounted시에만 호출
    load(){
      if(localStorage.getItem("canvasWidth") != null){
        this.canvasWidth = localStorage.getItem("canvasWidth")
      }
      if(localStorage.getItem("canvasHeight") != null){
        this.canvasHeight = localStorage.getItem("canvasHeight")
      }
      if(localStorage.getItem("polyLineWidth") != null){
        this.polyLineWidth = localStorage.getItem("polyLineWidth")
      }
    },
    uploadImage(){
      let imageTemp = this.$refs["imageFile"].files[0]
      const url = URL.createObjectURL(imageTemp)
      this.image = url
    },
    help(){
      let str = "정방향 샘플만을 위한 폴리 메이커 툴입니다."
      str += "\n"
      str += "\n"
      str += "1. 이미지를 업로드하시면 좌측 이미지 영역에 딱 맞춘 사이즈로 표시됩니다."
      str += "\n"
      str += "캔버스 사이즈가 자동 세팅되는데, 실제 이미지 사이즈가 아니므로 유의하셔야 합니다."
      str += "\n"
      str += "2. poly인풋추가를 선택하시면 poly를 한 개 더 그릴 수 있습니다."
      str += "\n"
      str += "poly는 length가 8개인 배열로 한정되며 배열 앞의 문자나, 배열 뒤의 문자(','포함)들은 trim해서 처리합니다."
      str += "\n"
      str += "예) ' asd[1  ,   1   ,   1  , \n50,50,50,50,1],' => [1,1,1,50,50,50,50,1]"
      str += "\n"
      str += "3. 비어있는 칸은 그리지 않습니다. 비어있지 않을 경우 에러메세지를 표시할 수 있습니다."
      str += "\n"
      

      alert(str)
    },
    addPoly(){
      this.polyColorJson.push([])
      this.polyStr.push([])
    },
    adaptPoly(){
      let info = []
      this.poly = this.getRealPoly(this.polyStr, info)
      let ctx = this.getContext()
      this.poly.forEach((el, index) => {
        this.drawSinglePoly(ctx, el, false, this.polyLineWidth, this.polyColor[index])
      })

      if(info.length != 0){
        alert(info + "번 인덱스 텍스트 박스의 양식이나 값에 문제가 있습니다. 확인해주세요.")
      }
    },
    getRealPoly(array, info){
      return array.map((el, index) => {
        if(el == null || el === '' || el.length == 0){
          return []
        }
        el = el.replace(/^[^[]*/, "").replace(/,\s*$/, "")
        let result = []
        try {
          result = JSON.parse(el)
        } catch (error) {
          console.log(`${index}번 인덱스 텍스트 박스 => `, error)
        }
        if(result.length != 8){
          info.push(index)
          result = []
        }
        return result
      })
    },
    mergePoly(){
      let len = this.checkedPoly.length
      let deltaX = 0
      let deltaY = 0
      for(let i=0; i< len ; i++){
        let arrayA = this.getRealPoly(this.polyStr[this.checkedPoly[i]], [])
        let arrayB = this.getRealPoly(this.polyStr[this.checkedPoly[(i+1) % len]], [])
        if(arrayA.length == 0 || arrayB.length == 0){
          return alert("병합할 폴리 값에 이상이 있습니다. 확인해주세요.")
        }
        for(let j=0; j< 4 ; j++){
          deltaX += arrayB[2*j] - arrayA[2*j]
          deltaY += arrayB[2*j + 1] - arrayA[2*j + 1]
        }
      }

      let resultArray = []
      if(deltaX >= deltaY){
        resultArray = this.mergeWider()
      }else if(deltaX < deltaY){
        resultArray = this.mergeHigher()
      }
      this.polyStr.push(JSON.stringify(resultArray))
    },
    
    clearPoly(){
      let ctx = this.getContext()
      ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height)
    },
    clearTexts(){
      this.polyStr = this.polyStr.map(() => "")
    },
    getContext(){
      const canvas = document.getElementById("canvas")
      return canvas.getContext("2d")
    },
    drawArrayPoly(context, array2D, fill, width, color) {
      if(array2D.length == 0){
        return
      }
      context.lineWidth = width;
      context.strokeStyle = color;
      context.fillStyle = color;
      array2D.forEach((el) => {
        if(el == null || el.length == 0){
          return
        }
        context.beginPath();
        context.moveTo(el[0], el[1]);
        context.lineTo(el[2], el[3]);
        context.lineTo(el[4], el[5]);
        context.lineTo(el[6], el[7]);
        if (fill) {
          context.fill();
        } else {
          context.closePath();
          context.stroke();
        }
      });
    },
    drawSinglePoly(context, array, fill, width, color){
      if(array.length == 0){
        return
      }
      context.lineWidth = width;
      context.strokeStyle = color;
      context.fillStyle = color;
      context.beginPath();
      context.moveTo(array[0], array[1]);
      context.lineTo(array[2], array[3]);
      context.lineTo(array[4], array[5]);
      context.lineTo(array[6], array[7]);
      if (fill) {
        context.fill();
      } else {
        context.closePath();
        context.stroke();
      }
    },
    onColorChange(values, index) {
      this.polyColor[index] = values.hex
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped src="../style/polyMaker.scss" lang="scss">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
