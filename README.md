# Angular_手繪功能

angular 手繪版功能


Angular+Rxjs

撰寫上為一碰到的問題是 Canvas 設定長寬要注意，若用css 去設定會發生畫筆與鼠標不同步的問題。


[線上demo](https://stackblitz.com/edit/angular-ivy-jgc3ph?file=src%2Fapp%2Fdraw%2Fdraw.component.ts)
```typescript


import { Component,  OnInit,} from '@angular/core';
import { fromEvent, switchMap,takeUntil, pairwise} from 'rxjs';

@Component({
  selector: 'app-draw',
  templateUrl: './draw.component.html',
  styleUrls: ['./draw.component.css']
})
export class DrawComponent implements OnInit {

  constructor() { }
  target:HTMLCanvasElement;
  ctx:CanvasRenderingContext2D;

  
  ngOnInit() {
    this.drawInit();
  }

  drawInit(){
    this.target = document.getElementById('draw') as HTMLCanvasElement;
    this.ctx  = this.target.getContext('2d');
    const rect = this.target.getBoundingClientRect();
    //高度寬度利用css設定， 會造成偏移
    let browserWidth = document.body.clientWidth;
    this.target.width=browserWidth;
    this.target.height=400;
    this.ctx.lineWidth = 5;
    this.ctx.lineCap = 'round';
    this.ctx.strokeStyle='#000000';
    this.ctx.lineWidth=2;
    
    const mouseDown$ = fromEvent<MouseEvent>(this.target, 'mousedown')
    const mouseMove$ = fromEvent<MouseEvent>(this.target, 'mousemove')
    const mouseUp$ = fromEvent<MouseEvent>(this.target, 'mouseup')
    const mouseOut$ = fromEvent<MouseEvent>(this.target, 'mouseout')

    const touchDown$ = fromEvent<TouchEvent>(this.target, 'touchstart')
    const touchMove$ = fromEvent<TouchEvent>(this.target, 'touchmove')
    const touchUp$ = fromEvent<TouchEvent>(this.target, 'touchend')

    //switchMap 在於將兩個事件流進行串接，上一個事件流若執行完不重要，可用switchMap
    touchDown$.pipe(
      switchMap(e => {
        return touchMove$.pipe(
          takeUntil(touchUp$),
          pairwise()//將前一個值與當前值射出
        )
        
      })
    ).subscribe(([preEvt, curEvt]) => {
      if (preEvt) {

        this.ctx.beginPath();
        let preTouch = preEvt.targetTouches[0];
        let curTouch = curEvt.targetTouches[0];

        let coords = this.relativeCoords(preEvt);
        this.ctx.moveTo(coords.x, coords.y)
        coords = this.relativeCoords(curEvt);
        this.ctx.lineTo(coords.x, coords.y)
        this.ctx.stroke()
      }
      
    });

    mouseDown$.pipe(
      switchMap(e=>{
        return mouseMove$.pipe(
          takeUntil(mouseUp$),
          takeUntil(mouseOut$),
          pairwise()//將前一個值與當前值射出
        )
      })
    ).subscribe(([preEvt, curEvt]) => {
      if (preEvt) {

        this.ctx.beginPath();
        let coords = this.relativeCoords(preEvt);
        this.ctx.moveTo(coords.x, coords.y)
        coords = this.relativeCoords(curEvt);
        this.ctx.lineTo(coords.x, coords.y)
        this.ctx.stroke()
      }
      
    });
  }
  private relativeCoords(event:TouchEvent|MouseEvent) {
    let x,y;
    let bounds = this.target.getBoundingClientRect();
    let isTouch =  event instanceof TouchEvent;
    if(isTouch){
      x = (<TouchEvent>event).touches[0].clientX - bounds.left;
      y = (<TouchEvent>event).touches[0].clientY - bounds.top;
    }else{
      x = (<MouseEvent>event).clientX - bounds.left;
      y = (<MouseEvent>event).clientY - bounds.top;
    }
    return { x, y };
  }
  clean(){
    this.ctx.clearRect(0, 0, this.target.width, this.target.height);
  }

}



```
