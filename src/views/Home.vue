<template lang="pug">
.v-container.pa-4
  // Main content
  #container(@wheel="onWheel($event)")
  v-layout.text-center(column, justify-center, align-center)

    v-flex.pt-4
      .caption
        router-link(to='/privacy') {{ $t("home.privacy") }}
</template>

<script lang="ts">
import Vue from 'vue'
import axios from 'axios'
import { loginFacebook, loginTelegram, loginGoogle } from '@/utils/api'
import Component from 'vue-class-component'
import { i18n } from '@/plugins/i18n'
import { namespace } from 'vuex-class'
import { User } from '@/models/User'
const { vueTelegramLogin } = require('vue-telegram-login')

import Konva from 'konva'
import VueKonva from 'vue-konva'
import { Canvas } from 'konva/lib/Canvas'

Vue.use(VueKonva)

const AppStore = namespace('AppStore')
const SnackbarStore = namespace('SnackbarStore')

// FB object is global, declaring here for TS
declare const FB: any

@Component({
  components: {
    vueTelegramLogin,
  },
})
export default class Home extends Vue {
  @AppStore.Mutation setUser!: (user: User) => void
  @SnackbarStore.Mutation setSnackbarError!: (error: string) => void

  configKonva = {
    width: 1920,
    height: 800,
    draggable: true,
    container: 'container',
    zoomable: true
  }
  configCircle = {
    x: 100,
    y: 100,
    radius: 70,
    fill: "red",
    stroke: "black",
    strokeWidth: 4
  }
  
  width = window.innerWidth
  height = window.innerHeight
  shadowOffset = 20
  tween = null
  blockSnapSize = 30

  shadowRectangle: any
  stage: any
  layer: any
  scaleBy = 2.5;

  newRectangle(x: number, y: number, layer, stage){
    let rectangle = new Konva.Rect({
      x: x,
      y: y,
      width: this.blockSnapSize * 6,
      height: this.blockSnapSize * 3,
      fill: '#fff',
      stroke: '#ddd',
      strokeWidth: 1,
      shadowColor: 'black',
      shadowBlur: 2,
      shadowOffset: {x : 1, y : 1},
      shadowOpacity: 0.4,
      draggable: true
    });
    rectangle.on('click', (e) => {
      document.body.style.cursor = '';
    })
    rectangle.on('mouseover', (e) => {
      console.log("hovered")

    })
    rectangle.on('dragstart', (e) => {
      this.shadowRectangle.show();
      this.shadowRectangle.moveToTop();
      rectangle.moveToTop();
    });
    rectangle.on('dragend', (e) => {
      rectangle.position({
        x: Math.round(rectangle.x() / this.blockSnapSize) * this.blockSnapSize,
        y: Math.round(rectangle.y() / this.blockSnapSize) * this.blockSnapSize
      });
      stage.batchDraw();
      this.shadowRectangle.hide();
    });
    rectangle.on('dragmove', (e) => {
      this.shadowRectangle.position({
        x: Math.round(rectangle.x() / this.blockSnapSize) * this.blockSnapSize,
        y: Math.round(rectangle.y() / this.blockSnapSize) * this.blockSnapSize
      });
      stage.batchDraw();
    });
    layer.add(rectangle);
  }

  animationDuration = 300  //in miliseconds
  startScale = 0
  startTime = 0
  endScale = 100
  startPosX = 0
  endPosX = 0
  startPosY = 0
  endPosY = 0

  onWheelAnimate(currentTime: number) {
    if (!this.startTime) this.startTime = Date.now();
    var elapsedTime = currentTime - this.startTime + .1;
    if (elapsedTime < this.animationDuration) {

        let currentValue = this.startScale + (elapsedTime / this.animationDuration) * (this.endScale - this.stage.scaleX());
        //set your field value to currentValue here with something like document.getElemenById...
        //call the next animation frame
        // console.log(currentValue);
        this.stage.scale({ x: currentValue, y: currentValue });
        let pointer = this.stage.getPointerPosition();

        let mousePointTo = {
          x: (pointer.x - this.stage.x()) / this.stage.scaleX(),
          y: (pointer.y - this.stage.y()) / this.stage.scaleY(),
        };
        // console.log(mousePointTo, pointer);
        var newPos = {
          x: pointer.x - mousePointTo.x * currentValue,
          y: pointer.y - mousePointTo.y * currentValue,
        };
        // this.newRectangle(mousePointTo.x * currentValue, mousePointTo.y * currentValue , this.layer, this.stage);

        // console.log("newPos", newPos);
        this.stage.position({
          x: (elapsedTime / this.animationDuration) * (this.endPosX - this.stage.x()),
          y: (elapsedTime / this.animationDuration) * (this.endPosY - this.stage.y())
        });
        setTimeout(() => this.onWheelAnimate(Date.now()), 1);
    }
  }


  onWheel(e: WheelEvent) {
    console.log(Date.now())
    this.startTime = 0;
    this.startScale = this.stage.scaleX();
    console.log(this.startScale)
    let pointer = this.stage.getPointerPosition();
    this.endScale = e.deltaY < 0 ? this.startScale * this.scaleBy : this.startScale / this.scaleBy;
    this.startPosX = pointer.x - this.stage.x()
    this.endPosX = this.stage.width() / 2
    this.startPosY = pointer.y - this.stage.y()
    this.endPosY = this.stage.height() / 2
    console.log(this.startPosX, this.startPosY)
    console.log(this.endPosX, this.endPosY)
    this.newRectangle(this.startPosX, this.startPosY, this.layer, this.stage);
    this.newRectangle(this.endPosX, this.endPosY, this.layer, this.stage);

    // this.stage.scale({x: this.endScale, y: this.endScale});

    console.log("->", this.endScale)
    this.onWheelAnimate(Date.now())

    this.stage.position({
      x: this.stage.x(),
      y: this.stage.y(),
    })
    this.stage.scale({x: this.endScale, y: this.endScale})
    this.stage.position({
      x: -this.stage.x(),
      y: -this.stage.y(),
    })

  }

  mounted() {

    this.shadowRectangle = new Konva.Rect({
      x: 0,
      y: 0,
      width: this.blockSnapSize * 6,
      height: this.blockSnapSize * 3,
      fill: '#FF7B17',
      opacity: 0.6,
      stroke: '#CF6412',
      strokeWidth: 3,
      dash: [20, 2]
    })

    this.stage = new Konva.Stage(this.configKonva);
    this.stage.on('click', (e) => {
      document.body.style.cursor = 'cursorurl';
    })

    var gridLayer = new Konva.Layer();
    var padding = this.blockSnapSize;
    console.log(this.width, padding, this.width / padding);
    for (var i = 0; i < this.width / padding; i++) {
      gridLayer.add(new Konva.Line({
        points: [Math.round(i * padding) + 0.5, 0, Math.round(i * padding) + 0.5, this.height],
        stroke: '#ddd',
        strokeWidth: 1,
      }));
    }

    gridLayer.add(new Konva.Line({points: [0,0,10,10]}));
    for (var j = 0; j < this.height / padding; j++) {
      gridLayer.add(new Konva.Line({
        points: [0, Math.round(j * padding), this.width, Math.round(j * padding)],
        stroke: '#ddd',
        strokeWidth: 0.5,
      }));
    }


    this.layer = new Konva.Layer();
    this.shadowRectangle.hide();
    this.layer.add(this.shadowRectangle);
    this.newRectangle(this.blockSnapSize * 3, this.blockSnapSize * 3, this.layer, this.stage);
    this.newRectangle(this.blockSnapSize * 10, this.blockSnapSize * 3, this.layer, this.stage);

    this.stage.add(gridLayer);
    this.stage.add(this.layer);
  }

}
</script>

<style>
.fb-signin-button {
  cursor: pointer;
  display: block;
  padding: 10px 46px;
  border-radius: 3px;
  background-color: #647daf;
  color: #fff;
  margin: 10px;
}
.g-signin-button {
  margin: 10px;
  cursor: pointer;
  display: block;
  padding: 10px 46px;
  border-radius: 3px;
  background-color: #ce5658;
  color: #fff;
}
</style>
