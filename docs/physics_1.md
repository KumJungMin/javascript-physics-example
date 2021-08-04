# physics.js의 기본을 알아보자
_모든 자료는 <a href="http://wellcaffeinated.net/PhysicsJS/">physics.js 공식문서</a>를 보고 작성하였습니다._

대부분의 물리엔진 라이브러리는 world, body, renderer, event 혹은 behavior 등으로 구성되어있다. <br/>
(사족을 붙이자면..) ~matter.js라는 물리엔진 라이브러리도 있는데 공식문서 설명도 너무 간단하고 <br/> 무엇보다 youtube에서 설명하는 공식 영상의 경우 p5.js를 같이 겸하는 등 
자체 라이브러리에 대한 설명이 부족하다... <br/> 그래서 개인적으로 physics.js 공식문서가 잘 작성되어 있어 시작하게 되었다;;~

<br/>


## 1. world 구축하기
- `world`는 말 그대로 세계이며, 우리가 만들 물리엔진을 구축하는 곳이다.
- 물리 엔진을 이용하여 시뮬레이션을 만들려면 제일 처음 선언해야 한다.
- 선언하는 방식은 아래와 같이 3가지 방법이 있다.
```jsx
//방법1
const world = Physics();
```

```jsx
//방법2
Physics(function(world){
    // use "world"
});

//방법3
Physics(function(world){
// use "world"
});
```
- 첫번째 방법이 간단해서 좋아보이지만, 2,3번째 방법이 전역 범위를 침해하지 않아 권장되는 스타일이다.
- 여러 월드를 만들어 사용하는 방법은 아래와 같다.

```jsx
// 각각의 시뮬레이션 월드 만들기
const ballSim = function(world){
    //ball 시뮬레이션 생성가능
}
const chainSim = function(world) {
  //chain 생성하기
}

//여러 시뮬레이션 월드 추가하기
const init = function() {
  const ballWorld = Physics(ballSim);
  const chainWorld = Physics(chainSim);
}
```

- world에서 여러가지 옵션을 사용할 수 있다.
- 옵션은 첫번째 인수로 전달하면 된다.

```jsx
Physics({
  timestep: 1000.0 / 160,  //시간 간격 설정
  maxIPF: 16,              //한 step당 최대 반복 횟수
  integrator: 'verlet'
}, function(world){
  //world 구축하기
})
```


<br/>


## 2. 시뮬레이션 작동시키기

- 한 프레임에서 시뮬레이션을 진행시키는 방법은 간단한다.
→ `step`메소드를 사용하여 현재 시간을 매개변수로 넘겨준다.
- 그런데 이 방법말고 `window.requestAnimationFrame`을 사용해 애니메이션 루프 내에서 시뮬레이션을 진행시키기도 한다.

<br/>

- PhysicsJS에서는 애니메이션 루프를 쉽게 하기 위해 `Physics.util.ticker`를 제공해준다!
- 이 `ticker`메소드는 requestAnimationFrame을 사용하므로 필요할 경우 polyfil 해야한다.
- 방법은 `ticker.on`을 사용해 이벤트를 구독하고, `start`로 진행시키면 된다.

```jsx
//ticker.on으로 구독하기
Physics.util.ticker.on(function(time) {
  world.step(time);  //월드의 step이 현재 시간을 따라 진행
})

//ticker 시작하기
Physics.util.ticker.start();
```

- `add()`를 이용하여 여러가지 것(몸체, 행동, 랜더러 등)을 추가할 수 있다.
- 여러 개의 몸체와 행동을 한 세계에 추가할 수는 있다.
- 그러나 하나의 세계에 하나의 렌더러(`renderer`)와 하나의 통합자(`integrator`)만 추가할 수 있다.

<br/>


## 3. bodies, 사물 객체 만들기

- 원, 포인트 등 가능한 여러가지 것들을 만들 수 있다.
- `Physics.body`를 사용해 이 객체들을 만들 수 있다.

```jsx
//원 사물 객체 만들기
const ball = Physics.body("circle", {
  x: 50,    //x좌표
  y: 30,    //y좌표
  vx: 0.2   //x방향으로 속도
  vy: 0.01 //y방향으로 속도
  radius: 20, //반지름
})

//world에 원 추가하기
world.add(ball);
```

<br/>


## 4. behavior(동작) 정의하기

- `behavior`(동작)은 `world`에 적용할 물리 규칙을 정의한다.
- 만약 이 동작을 정의하지 않는다면, 그 `world`는 마찰이 없는 무한한 진공 상태가 된다.
- 이 동작으로 우리는 중력이나, 충돌 등을 정의할 수 있다.
- 중력의 경우 y방향으로의 일정한 가속도(`constant-acceleration`)를 설정하면 된다.

```jsx
//중력 정의하기
const gravity = Physics.behavior('constant-acceleration', {
  acc: {x:0, y:0.0004}. //기본값
})

// world에 중력이라는 동작을 추가하기
world.add(gravity);   
```

- 충돌과 같은 동작의 경우, 해당 객체들의 동작에 영향을 주진 않지만 충돌한 객체들의 정보를 추출할 수 있다. 
- 이처럼 동작은 사물 객체의 움직임을 제어하기도 하지만 정보를 추출하는 추출기 역할을 할 수 있다.

<br/>

## 5. 랜더러 만들기

- 랜더러는 시뮬레이션의 `viewport`(사용자에게 보여지는 영역)를 정의한다.
- `Dom`, `html canvas` 랜더링을 위한 랜더러도 있는데, 이 경우 [커스텀 랜더러](https://github.com/wellcaffeinated/PhysicsJS/wiki/Renderers)를 만들어야할 가능성이 높지만, 어렵지 않다!

```jsx
const renderer = Physics.renderer('canvas', {
    el: 'element-id',
    width: 500,
    height: 300,
    meta: false, // don't display meta data
    styles: {
        // set colors for the circle bodies
        'circle' : {
            strokeStyle: 'hsla(60, 37%, 17%, 1)',
            lineWidth: 1,
            fillStyle: 'hsla(60, 37%, 57%, 0.8)',
            angleIndicator: 'hsla(60, 37%, 17%, 0.4)'
        }
    }
});
// add the renderer
world.add( renderer );
```

<br/>

- `world`의 상태를 캔버스에 랜더링하려면, 모든 단계(step)에서 `world.render` 메소드를 호출해야한다.
- 마치 world.step → world.render 순서로 호출하는 것과 유사하다.

```jsx
var renderer = Physics.renderer('canvas', {
    //...
});
// add the renderer
world.add( renderer );
world.on('step', function(){
   //step 마다 render를 실행시켜야함
    world.render();
});
```

<br/>

## 6. integrators로 수치적 설정하기

- `integrators`(적분기)는 body의 물리적 특성을 수치적으로 통합한다. 

```jsx
Physics({
    integrator: 'verlet'
}, function(world){
    // set up...
});

// equivalent to...
Physics(function(world){
    world.add( Physics.integrator('verlet') );
    // set up...
});
```

<br/>

## 7. 전체적인 코드 예시보기

```jsx
//예시
Physics(function(world){
  //1.보여지는 영역의 높이, 너비 설정
  var viewWidth = 500;
  var viewHeight = 300;

  //2.뷰포트 설정
  var renderer = Physics.renderer('canvas', {
    el: 'viewport',
    width: viewWidth,
    height: viewHeight,
    meta: false, // don't display meta data
    styles: {
        // set colors for the circle bodies
        'circle' : {
            strokeStyle: '#351024',
            lineWidth: 1,
            fillStyle: '#d33682',
            angleIndicator: '#351024'
        }
    }
  });

  //3. 랜더리 추가하기
  world.add( renderer );
  //4. 각 step마다 랜더링하기
  world.on('step', function(){
    world.render();
  });

  // 5.bounds of the window
  var viewportBounds = Physics.aabb(0, 0, viewWidth, viewHeight);

  // 6. constrain objects to these bounds
  world.add(Physics.behavior('edge-collision-detection', {
      aabb: viewportBounds,
      restitution: 0.99,
      cof: 0.99
  }));

  // 7. add a circle
  world.add(
      Physics.body('circle', {
        x: 50, // x-coordinate
        y: 30, // y-coordinate
        vx: 0.2, // velocity in x-direction
        vy: 0.01, // velocity in y-direction
        radius: 20
      })
  );

  // 8. ensure objects bounce when edge collision is detected
  world.add( Physics.behavior('body-impulse-response') );

  // 9. add some gravity
  world.add( Physics.behavior('constant-acceleration') );

  // 10. subscribe to ticker to advance the simulation
  Physics.util.ticker.on(function( time, dt ){

      world.step( time );
  });

  // 11. start the ticker
  Physics.util.ticker.start();

});
```

---
